---
layout: sidenote
title:  "How to Enable the Traefik Dashboard"
url: /run-traefik-dashboard
comments: true
date: 2020-12-9
categories: side-notes
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
show_related_posts: false
feature_image: feature-sidenotes
---
{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/Traefik_dashboard.png" description="Traefik Dashboard" %}

The Traefik ingress controller is included out-of-the-box with Rancher k3s. It deploys into the kube-system namespace. Take a moment to have a look at the kube-system namespace.

```shell
kubectl -n kube-system get all
```
{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/kube-system-namespace.png" description="Traefik System Namespace" %}

While the service, “service/traefik,” is running, the dashboard is not. To enable the dashboard, edit the config map. The name of the config map is specified in the Traefik deployment at Volumes → config → name.

```shell
kubectl -n kube.system describe deploy traefik
```
{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/traefik-config-map.png" description="Section of the Traefik Deployment YAML" %}

Edit the “traefik” config map and add a section named “api” with an attribute of “dashboard” equal to true.

```shell
kubectl -n kube-system edit cm traefik
```
{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/enable-dashboard.png" description="Enable the Traefik Dashboard" %}

Bounce Traefik to pick up the updated config map by scaling it down to zero, then back up to one.

```shell
kubectl -n kube-system scale deploy traefik --replicas 0
kubectl -n kube-system scale deploy traefik --replicas 1
```
{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/bounce_traefik.png" description="Bounce the Traefik Pods" %}

Start port forwarding to the Traefik service.

{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/forward_to_deployment.png" description="Forward to the Traefik Service" %}

```shell
export KUBECONFIG=~/kubeconfig
kubectl -n kube-system port-forward deployment/traefik 8080 &
kubectl proxy &
```
{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/traefik_port_forwarding.png" description="Port Forwarding" %}


Finally, open the dashboard at <a href="http://localhost:8080/dashboard">http://localhost:8080/dashboard</a>.

{% include image.html url="/img/post-assets/sidenote-traefik-dashboard/Traefik_dashboard.png" description="Traefik Dashboard" %}

____
# References
[Rancher K3S Ingress Demo with Traefik](https://www.youtube.com/watch?v=12taKl5iCpA)





