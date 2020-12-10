---
layout: post
title:  "Kubernetes Networks and Ingress"
url: /k8s-network-ingress
comments: true
date: 2020-12-9 14:28:00
categories: kubernetes raspberry
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 5
show_related_posts: false
feature_image: feature-kubernetes
square_related: recommend-kubernetes
---
I [assembled a Raspberry Pi cluster](/some-assembly-required) previously in this series, [installed Ubuntu Groovy Gorilla](/running-ubuntu-on-rpi), and [installed Rancher k3s Kubernetes](/running-kubernetes-on-rpi). It’s close to running interesting workloads, but first, it needs ingress and storage volumes. Ingress is not difficult, but this being a learning lab, let’s sharpen our understanding of Kubernetes networking. 

# Basic Kubernetes Network Concepts

This diagram from the [Rancher K3s web site](https://k3s.io/) shows the core components of Rancher K3s Kubernetes.

{% include image.html url="/img/post-assets/2020-12-9-ingress-controller/k3s-diagram.png" description="Rancher k3s Components" %}

It shows a network Tunnel Proxy connecting the nodes and the Kube Proxy, which we [used to connect to the Kubernetes Dashboard](/sidenote-dashboards) in an earlier post. You also see [Flannel](https://github.com/coreos/flannel), containers, and pods. These need further exploration.

{% include image.html url="/img/post-assets/2020-12-9-ingress-controller/matryoshka_dolls.png" description="Kubernetes Matryoshka Dolls" %}

If you run in the cloud or on a virtual machine, you have even more layers than what is shown above. In my case, the host is a Raspberry Pi running a Kubernetes worker node. The worker node runs the Flannel daemon to supply the overlay network, and the container runtime, [containerd](https://containerd.io/). Flannel and containerd provide the foundation for running pods. Each pod includes one or more containers for applications.

The Kubernetes specification includes network requirements but allows flexibility in the implementation. Some core network tenants are these:
* Pods on a node can communicate with all pods on all nodes without NAT.
* Agents on a node can communicate with all pods on that node.
* Pods in the host network of a node can communicate with all pods on all nodes without NAT.

Rancher k3s Kubernetes uses Flannel, by [CoreOS](https://coreos.com/), for its CNI (Container Network Interface), but many other CNIs exist. See Rancher’s article [Comparing Kubernetes CNI Providers: Flannel, Calico, Canal, and Weave](https://rancher.com/blog/2019/2019-03-21-comparing-kubernetes-cni-providers-flannel-calico-canal-and-weave).

Take a closer look at the network.

{% include image.html url="/img/post-assets/2020-12-9-ingress-controller/Flannel_Overlay.png" description="Flannel Overlay Network" %}

The DHCP IP addresses on my home network start at 192.168.1.100, leaving IP addresses 192.168.1.2 through 192.168.1.99 available for static IP addresses. The figure above depicts two IPs from my range of static IPs assigned to Raspberry Pis, 192.168.1.10 and 192.168.1.20.

## Flannel

Flannel is a layer-3 network fabric designed for Kubernetes. It configures an extensive internal overlay network that spans across every node within the cluster. In the illustration above, Flannel assigned my two Raspberry Pis node ranges of 10.42.1.0/16 and 10.42.2.0/16, respectively. The container runtime on the left allocated IP addresses 10.42.1.2 and 10.42.1.3 to its pods. On the right, the container runtime assigned the IP addresses 10.42.2.2 and 10.42.2.3.

The arrow between pods 10.42.1.3 and 10.42.2.2 represents traffic between nodes over the Tunnel Proxy. Of course, pods on the same node can send network traffic to each other, but since pods are ephemeral, keeping track of the pod addresses is problematic. That is where services come into play.

# Services
Service is an abstraction for a set of pods. Services insulate the service clients from the details of where pods run.

## Service Types
The four service types are cluster IP, external name, node port, and load balancer.

### Cluster IP
A cluster IP service is the default Kubernetes service type. It allows other apps in the cluster to communicate with the service but does not allow direct external traffic. However, indirect external traffic is possible through ingress routing rules.

### Node Port
A node port service is the most direct route to allow traffic into a cluster. The Kubernetes control panel allocates a port to the service, which is proxied by Kube Proxy on all the nodes, thus directing traffic to the service port no matter at which node the traffic arrives. By default, Kubernetes assigns the service a port number between 30,000 and 32,767.

### External Name
The external name type allows you to access services that are not running in your cluster, acting as if they are running in your cluster. It maps the service to the contents of the “externalName” field (e.g. foo.bar.example.com). It has no proxying of any kind.

### Load Balancer
Load balancer services use an external load balance provided by a cloud provider. A bare-metal Kubernetes cluster can use the load balancer service type with [MetalLB](https://metallb.universe.tf/), a product built to offer external load balancing to Kubernetes not hosted on a cloud provider.

## Kube Proxy
Kube Proxy is a network proxy that runs on every node of the Kubernetes cluster. It maintains the network rules for the nodes, allowing network communication with the pods. Kube Proxy has many options for routing traffic to pods in a replica set, like round-robin, beyond this post’s scope. 

{% include image.html url="/img/post-assets/2020-12-9-ingress-controller/service_proxy.png" description="Services and KubeProxy" %}

The illustration above depicts a cluster IP service called by one or more client applications in the cluster. It also has an ingress routing rule allowing an external API Gateway to reach the service’s backend pods.

# Ingress Controller
Now that we have our network homework out of the way, rather than include an exhaustive tutorial in this post, there are two side notes linked below: one for enabling the Traefik Dashboard and one on exposing a cluster IP service with an ingress rule. 

As mentioned at the start of this post, the ingress setup is straightforward: 

1. Create an application deployment (the side note deploys the Nginx Docker image).
1. Create a service (the side note uses the cluster IP service type, but it could be service type node port)
1. Create an ingress routing rule. (The side note uses the deployment from the k3s GitHub guide)

Here are the step-by-step guides, if you are interested.

{% include tip.html content=" Learn how to enable the Traefik Dashboard: [Jump to the article](/sidenote-run-traefik-dashboard)." %}

{% include tip.html content="Learn how to expose a service with the Traefik ingress controller:  [Jump to the article](sidenote-expose-traefik-ingress)." %}

Stay tuned for the next post about persistent storage on Raspberry Pi NAS using a 1Gb Samsung EVO SATA M.2.

{% include image.html url="/img/post-assets/2020-12-9-ingress-controller/preview_nas.png" description="Raspberry Pi NAS" %}

----
# References
* [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
* [Kubernetes Ingress - Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/)
* [Connecting Applications with Services](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)
* [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
* [Flannel is a network fabric for containers, designed for Kubernetes](https://github.com/coreos/flannel)
* [k3d/exposing_services.md at main · rancher/k3d · GitHub](https://github.com/rancher/k3d/blob/main/docs/usage/guides/exposing_services.md)





