---
layout: sidenote
title:  "How to Expose a Service with Traefik"
url: /expose-traefik-ingress
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
This post walks through exposing a service with Traefik ingress. 

{% include image.html url="/img/post-assets/sidenote-traefik-ingress/welcome-to-nginx.png" description="Nginx Welcome Page" %}

# Create the Namespace
First, create a namespace for this example.

```shell
kubectl create namespace sample-nginx
```

# Deploy the Sample Nginx Application
Create the deployment YAML for Nginx. The deployment name is “sample-nginx-deployment” with an exposed container port of 80.

```shell
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-nginx-deployment
  namespace: sample-nginx
spec:
  selector:
    matchLabels:
      run: sample-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: sample-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```
<figcaption class="image-caption">sample-nginx-deployment.yaml</figcaption>

Deploy the application to Kubernetes.

```shell
kubectl apply -f sample-nginx-deployment.yaml
```

Inspect the deployment.
```shell
kubectl describe deployment sample-nginx-deployment -n sample-nginx
```
{% include image.html url="/img/post-assets/sidenote-traefik-ingress/describe_sample_deployment.png" description="Deployment Description" %}

Verify the pods.
```shell
kubectl get pods -n sample-nginx -o wide
```
{% include image.html url="/img/post-assets/sidenote-traefik-ingress/get_sample_nginx_pods.png" description="Get Pods" %}

# Add a Service
Create the YAML for the service.

```shell
---
apiVersion: v1
kind: Service
metadata:
  name: sample-nginx
  namespace: sample-nginx
  labels:
    run: sample-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: sample-nginx
```
<figcaption class="image-caption">sample-nginx-service-deployment.yaml</figcaption>

Apply the service YAML.

```shell
kubectl apply -f sample-nginx-service-deployment.yaml
```

Verify the service.

```shell
kubectl get svc -n sample-nginx
```
{% include image.html url="/img/post-assets/sidenote-traefik-ingress/get_svc.png" description="Get Service" %}

# Add the Ingress

Create the Ingress YAML. 

```shell
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-nginx-ingress
  namespace: sample-nginx
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: byteworksinc.com
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sample-nginx
            port:
              number: 80
```
<figcaption class="image-caption">Sample-nginx-ingress-deployment.yaml</figcaption>

The “host” attribute is optional. Most demonstrations use a host of “example.com.” If you keep the host attribute, you need to edit your /etc/hosts files to route that domain to the cluster. If you leave it out, you can open a web browser directly to any of the nodes, for example, http://192.168.1.10, in my case.

Since I set “host” to “byteworksinc.com” in my ingress yaml file, I updated my host file to point byteworkinc.com to the Kubernetes cluster.

```shell
echo 192.168.1.10 byteworksinc.com | sudo tee -a /etc/hosts
cat /etc/hosts | tail -n 1
```
{% include image.html url="/img/post-assets/sidenote-traefik-ingress/etc_hosts.png" description="Confirm /etc/hosts Change" %}

Apply the ingress YAML file.

```shell
kubectl apply -f sample-nginx-ingress-deployment.yaml
```

Refresh the Traefik Dashboard to see the updated information.

{% include image.html url="/img/post-assets/sidenote-traefik-ingress/updated-traefik-dashboard.png" description="Updated Traefik Dashboard" %}

The frontend shows the root context rule “/.” The backend lists the two nodes where Nginx is running.

Open a web browser to http://byteworksinc.com to see if the default Nginx page opens.

{% include image.html url="/img/post-assets/sidenote-traefik-ingress/welcome-to-nginx.png" description="Nginx Welcome Page" %}

The Traefik ingress rule is working as intended.

----
# References
[k3d/exposing_services.md at main · rancher/k3d · GitHub](https://github.com/rancher/k3d/blob/main/docs/usage/guides/exposing_services.md)
[Create a Kubernetes TLS Ingress from scratch in Minikube](https://www.youtube.com/watch?v=7K0gAYmWWho)
<a href="https://github.com/prometheus-operator/prometheus-operator/issues/800#issuecomment-349439795" >https://github.com/prometheus-operator/prometheus-operator/issues/800#issuecomment-349439795</a>

