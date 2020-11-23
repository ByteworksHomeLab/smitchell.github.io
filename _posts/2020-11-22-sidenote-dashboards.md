---
layout: sidenote
title:  "How to use Kubernetes Dashboard"
url: /how-to-use-k8s-dashboard
comments: true
date: 2020-11-22
categories: side-notes
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
show_related_posts: false
feature_image: feature-sidenotes
square_related: recommend-sidenotes
k8dash: u-1jGAhAHAM
kubevious: YVBjt-9ugTg
lens: 04v2ODsmtIs
octant: jUuZxgjyPPc
---
{% include image.html url="/img/post-assets/sidenote-dashboard/KubernetesDashboard.png" description="Kubernetes Dashboard" %}

<a href="https://github.com/kubernetes/dashboard">Kubernetes Dashboard</a> is the web UI that ships with Kubernetes. That is the dashboard we install in this post using kubectl proxy and a token, without using an Ingress controller or authenticating proxy.

## Alternative Dashboards

### K8Dash
<a href="https://github.com/indeedeng/k8dash">K8Dash</a> is a popular open-source alternative to Kubernetes Dashboard. K8Dash must be exposed publicly on the cluster, requiring an Ingress controller and an authenticating proxy. 
{% include youtubePlayer.html id=page.k8dash %}

### Kubevious
<a href="https://kubevious.io/">Kubevious</a> is a read-only tool for examining Kubernetes clusters.
{% include youtubePlayer.html id=page.kubevious %}

### Octant
<a href="https://octant.dev/">Octant</a> is a developer-centric, read-only Kubernetes UI.
{% include youtubePlayer.html id=page.octant %}

### Lens
<a href="https://k8slens.dev/">Lens</a> is a desktop application built as a Kubernetes IDE. It is free, easy to install, and automatically reads the Kubeconfig file on your system. 
{% include youtubePlayer.html id=page.lens %}

### Metrics Service
Most Kubernetes Dashboards give you access to metrics if you install the Metrics Service, described here: <a href="https://github.com/kubernetes-sigs/metrics-server">Kubernetes Metrics Server</a>.

## Installation
Start by following Rancher’s dashboard instructions: <a href="https://rancher.com/docs/k3s/latest/en/installation/kube-dashboard/">Rancher Docs: Kubernetes Dashboard</a>.
  
Verify that the Kubernetes Dashboard pod is running.

```shell
```shell
pi@pi1:~ $ sudo kubectl get pods --namespace kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-7b59f7d4df-724pf   1/1     Running   0          12m
kubernetes-dashboard-665f4c5ff-5pbl2         1/1     Running   0          12m
```

Note the node name above, "kubernetes-dashboard-xxxxxxxxxx-xxxxx." Yours will be different. You need the name to set-up the kubectl proxy from your desktop.

Verify that the Kubernetes Dashboard Services are running.

```shell
sudo kubectl get services --namespace kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes-dashboard        ClusterIP   10.43.135.238   <none>        443/TCP    14m
dashboard-metrics-scraper   ClusterIP   10.43.210.211   <none>        8000/TCP   14m
```

Congratulations! You now have a working Kubernetes Dashboard. To access the Dashboard from your computer, set up the port forwarding and start the kubeclt proxy, substituting the pod name that you listed above.

```shell
export KUBECONFIG=~/kubeconfig
kubectl --namespace=kubernetes-dashboard port-forward kubernetes-dashboard-665f4c5ff-5pbl2 8443 &
Forwarding from 127.0.0.1:8443 -> 8443
kubectl proxy &
Starting to serve on 127.0.0.1:8001 
```

Open a browser and navigate to the Kubernetes Dashboard. Use the admin-user bearer token from above to log in. 

{% include image.html url="/img/post-assets/sidenote-dashboard/DashboardLogin.png" description="Dashboard Login with Token" %}

Navigate to the Nodes page to see the agent nodes you just installed.

You can put an authenticating proxy in front of the Kubernetes Dashboard if you don’t want to use kubectl proxy and a token, but that is beyond the scope of this post. See Joe Beda’s post, “<a href="https://blog.heptio.com/on-securing-the-kubernetes-dashboard-16b09b1b7aca">On Securing the Kubernetes Dashboard</a>.”

----
## References
* <a href="https://github.com/kubernetes-sigs/metrics-server">Cluster-wide aggregator of resource usage data</a>
* <a href="https://rancher.com/docs/k3s/latest/en/installation/kube-dashboard/">Rancher Docs: Kubernetes Dashboard</a>
* <a href="https://blog.heptio.com/on-securing-the-kubernetes-dashboard-16b09b1b7aca">On Securing the Kubernetes Dashboard | by Joe Beda</a>
* <a href="https://blog.heptio.com/how-to-deploy-web-applications-on-kubernetes-with-heptio-contour-and-lets-encrypt-d58efbad9f56">How-To: Deploy web applications on Kubernetes with Heptio Contour and Let’s Encrypt</a>











