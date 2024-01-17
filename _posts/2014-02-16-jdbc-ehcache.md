---
layout: post
title:  "Spring JDBC and Ehcache"
url: /spring-jdbc-and-ehcache
comments: true
date: 2014-02-16 15:30:03
categories: spring architecture
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
feature_image: feature-architecture
show_related_posts: false
square_related: recommend-raspberry
youtubeid: lW3zlj3zWjM
---
I refactored a client's code to meet current standards a few months ago. The client had three projects talking to the same database in three different ways: MyBatis, Hibernate, and Spring JDBC. Also, two of the projects were still using ANT. I switched all the projects to Maven, Spring, and a shared DAO layer using Spring JDBC.

One of the client's projects exports millions of rows of data. Small changes can significantly affect export time. One heavily called block of code used a HashMap in an instance variable to cache some objects. I decided to move caching into the DAO instead. That change caused the export to run three times longer than before. Why was Ehcache so slow? Had I done something wrong?

## Cache Performance
I set up a test project to measure how long it took to read 10,000 rows from a 340-row table randomly. Here are the results.
{% include image.html url="/img/post-assets/2014-02-16-jdbc-ehcache/chart_1.png" description="Spring ehcache performance" %}
It took 0.043 seconds to perform 10,000 random reads from a HashMap of 340 objects. By contrast, it took almost 27 seconds to fetch the same set of 10,000 random keys directly from MySQL. The H2 database only took 0.58 seconds. The elapsed times were about the same using Ehcache backed by either H2 or MySQL, taking 2.36 seconds and 2.42 seconds, respectively.

Example Project on GitHub

I created a simple test project with H2 Database and MySQL to experiment with my configuration. You can find the code on GitHub to run the tests yourself.

## Setting up Spring JDBC and Ehcache

I used annotations to set-up Spring JDBC and Ehcache.

Here is how I set-up the cache in the Spring context.

```shell
    <bean id="ehcache" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean"
          p:configLocation='ehcache.xml'
          p:shared="true" />
    <bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager" p:cacheManager-ref="ehcache" />
    <cache:annotation-driven />
```

Here is an example from the Country DAO in the sample project.

@Repository
public class CountryDaoImpl implements CountryDao {

```shell
    @Cacheable("countries")
    public Country findByPrimaryKey(final Long id) {
        final MapSqlParameterSource params = new MapSqlParameterSource("value", id);
        final String sql = selectSQL.concat(" where ").concat(CountryColumn.COUNTRY_ID.name()).concat(" = :value");
        final List results = jdbcTemplate.query(sql, params, countryRowMapper);
        if (results.size() > 0) {
            return results.get(0);
        }
        return null;
    }

    @CacheEvict(value = "countries", allEntries=true)
    public void save(Country instance) {
        final Date now = new Date();
        instance.setLastUpdateDate(now);
        if (instance.getId() ==  null) {
            instance.setCreateDate(now);
            insert(instance);
        } else {
            update(instance);
        }
    }
...
}
```

The Ehcache configuration was equally straightforward.

```shell
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="true" monitoring="autodetect" dynamicConfig="true">
    <diskStore path="java.io.tmpdir"/>
    <defaultCache maxEntriesLocalHeap="10000" eternal="false" timeToIdleSeconds="120" timeToLiveSeconds="120"
                  diskSpoolBufferSizeMB="30" maxEntriesLocalDisk="10000000" diskExpiryThreadIntervalSeconds="120"
                  memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
    <cache name='countries' maxElementsInMemory='400' eternal='true' overflowToDisk='false' />

</ehcache>
```

## Conclusions

I'm still figuring out how best to leverage Ehcache, but it is clear that for small collections of simple objects, like categories, types, countries, states, roles, etc, Ehcache is overkill. For those types of objects, I will add a HashMap to my service object to act as a simple cache.

The place for Ehcache will probably turn out to be those compound object graphs assembled from calls to multiple tables. I'm going selectively leverage Ehcache in my service (@Component) implementations to hang on to those complex, composite objects.

If I succeed, I'll add a follow-up to this post.

