---
layout: post
title:  "Cloud-native Python from a Spring Developer Perspective"
url: /python-microservices
comments: true
date: 2020-11-12 11:13:00
categories: python kubernetes
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 10
show_related_posts: false
feature_image: feature-python
square_related: recommend-python
---
When a client asked for microservices written in Python, instead of the Spring Cloud ecosystem that I’m accustomed to, it was an adjustment. The client wanted to replace an aging desktop application used by several hundred support staff with a simple, web-based application written in a programming language that the team knew. 

These are my takeaways from the project:

* Good microservice design transcends programming languages.
* REST and ORM libraries written in different languages are easy to transition between for experienced developers.
* JOSE (Javascript Object Signing and Encryption) simplifies securing resource servers.
* NATS is a simpler alternative to messaging services like RabbitMQ.

During design, we chose microservice boundaries that aligned with the application’s backing services. This system aggregates data from Active Directory, NetIQ, and UltiPro, so we built a microservice for each of them. We could have created fine-grained microservices for individual domain contexts, like a facility service, audit service, keywords service, etc.. Given the project budget and the type of application we were replacing, one microservice per backend made the most business sense.

Aside from the syntactical differences between Java and Python, the overall project semantics felt familiar, making for an easy transition. Instead of Spring Web MVC, we used Python Flask, and instead of Spring Data JPA, we used Marshmallow with SQLAlchemy. Rather than having Spring REST Docs and annotation-driven Swagger files, we hand-coded the Swagger YAML files. Python libraries exist that generate Swagger files, but we didn’t have time to add them. We loaded the Swagger file into ng OpenAPI Gen to generate the Angular API clients, something I never tried before. I recommend it. 

As for security, the client uses Ping Identity for single-sign-on, so we didn't have to worry about standing up an authentication service. The authentication process didn’t require an OAuth2 Python library. We built a simple Security microservice to interface with Ping Identity. The frontend redirects the user to Ping Identity to do OpenID Connect single-sign-on. The authorization code returned by Ping Identity is then posted to the Security microservice via an API Gateway to exchange with Ping Identity for an access token. 

Instead of setting up the API Gateway as an OAuth2 Resource Service using Spring Security, we use Python-JOSE to verify the access token. The frontend caches the access token in local storage and passes it in the Authorization header whenever it calls the API Gateway. The Security microservice passes that access token along with the JWT signing key, fetched from the Ping Identity JWKS endpoint at startup, into the Python-JOSE decode method to confirm the access token’s authenticity. This approach to token verification works much faster than calling the Ping Identity token introspection endpoint for validation.

As far as not having Spring Discovery and Spring Configuration goes, Kubernetes Services and Namespaces work just as well. Load balanced Kubernetes services allow the client to scale microservices up and down, and the Service load balancer knows when instances get added or removed. Service registration is outside of the application.

Using separate Kubernetes namespaces for development and production replaced the need for a configuration service. The release pipeline supplies the appropriate variables and secrets for a given target namespace. Also, having separate namespaces allowed us to reuse the Kubernetes Service URLs in both environments, so the API Gateway calls the same backend microservice URLs in development as it does in production.

System ingress from the client’s Intranet is limited to Nginx, which hosts frontend resources, and to the API Gateway service. The backend services connect to a separate subnet to which only the API Gateway has access. The backend services can all communicate with NATS, used for the event-driven design, which connects to the same subnet.

{% include image.html url="/img/post-assets/2020-11-12-python-microservices/python_microservices.png" description="Kubernetes Diagram" %}


The NATS messaging service took minimal work to configure. The original desktop application had a delay while it logged each user action. With the new system, publishing user actions to NATS happens very fast, and the time it takes to write the event to the audit log happens in the background. Also, changing LDAP data was very slow in the desktop application. The client was thrilled with our asynchronous design for LDAP changes. Users get a sub-second response time when they publish change requests to NATS, and then they continue working while the updates complete behind the scenes.

One thing we didn’t get to is a distributed cache. There are many auto-completed input fields and lists that use infrequently changed LDAP data, such as facilities, departments, positions, and job roles. The web interface would be snappier if it used an in-memory cache, like Redis. 

Overall, the project was a success, thanks in large part to my teammates at Ahead. We delivered the simplified, microservice solution that the client asked for written in the client’s preferred programming language.

# References
* <a href="https://www.behance.net/jessjwilliamson/">Python Logo Credit: Jessica Williamson</a>
* <a href="https://www.thinkahead.com/">Ahead</a>
* <a href="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services">Active Directory</a>
* <a href="https://flask.palletsprojects.com/en/1.1.x/">Flask</a>
* <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/">Kubernetes Namespace</a>
* <a href="https://www.nginx.com/products/nginx/kubernetes-ingress-controller/">Kubernetes Nginx</a>
* <a href="https://kubernetes.io/docs/concepts/services-networking/service/">Kubernetes Service</a>
* <a href="https://nats.io/">nats.io</a>
* <a href="https://www.microfocus.com/en-us/products/netiq/overview">NetIQ</a>
* <a href="https://www.npmjs.com/package/ng-openapi-gen">ng OpenAPI Gen</a>
* <a href="https://www.pingidentity.com/">Ping Identity</a>
* <a href="https://python-jose.readthedocs.io/en/latest/">Python JOSE</a>
* <a href="https://spring.io/projects/spring-cloud">Spring Cloud</a>
* <a href="https://spring.io/projects/spring-data-jpa">Spring Data JPA</a>
* <a href="https://spring.io/projects/spring-restdocs">Spring REST Docs</a>
* <a href="https://spring.io/projects/spring-security">Spring Security</a>
* <a href="https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#spring-web">Spring Web MVC</a>
* <a href="https://www.sqlalchemy.org/">SQLAlchemy</a>
* <a href="https://www.ultimatesoftware.com/UltiPro-Services">UltiPro Services</a>

