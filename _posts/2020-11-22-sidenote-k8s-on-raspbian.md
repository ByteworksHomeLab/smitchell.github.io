---
layout: sidenote
title:  "How to Install Rancher Labs k3s on Raspbian"
url: /how-to-install-k3s-on-raspbian
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
---
{% include image.html url="/img/post-assets/sidenode-k8s-on-raspbian/k8s_on_raspbian.png" description="Kubernetes on Raspbian" %}
# Preparing Raspbian
Before jumping into Kubernetes, if you run Raspbian instead of Ubuntu, it needs to be setup for virtualization. If you are running Ubuntu, skip to "Installing the Master Node."

{% include tip.html content="Read how I made the switch from 32-bit Raspbian to 64-bit Ubuntu to improve Docker image compatibility. <a href='/running-ubuntu-on-rpi'>Jump to Ubuntu Post.</a>" %}

Containers and virtual machines rely on Linux namespaces and control groups for security, so we need to enable them in Raspbian by editing the /boot/cmdline.txt file. Reboot the nodes when you finish.

```shell
sudo cp /boot/cmdline.txt /boot/cmdline.txt.bak1
orig="$(head -n1 /boot/cmdline.txt) cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory"
echo $orig | sudo tee /boot/cmdline.txt
sudo reboot
```
{% include warning.html content="If you forget this step, the k3s agent won’t work. I build all my nodes manually, and I have accidentally skipped this step. When I do, the k3s agent appears to install correctly, but it never shows up with “kubectl get nodes.” Digging into the logs you find that the agent won’t start without the Linux CPU and memory control groups." %}


# Installing the Master Node

It is straightforward to get started with K3s. See the <a href="https://k3s.io/">Quick Start Guide</a>. I installed k3s slightly differently using the command below. It downloads an installation script, pipes it into the shell to install and configure the master node, and then starts the server. The Kubeconfig config is stored in /etc/rancher/k3s/k3s.yaml during the process.

```shell
curl -sfL https://get.k3s.io | sh -s - server &
```

When the server starts, verify the installation with this command:

```shell
$ sudo k3s kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
pi1    Ready    master   3m15s   v1.19.3+k3s3
```

Next, copy the kubeconfig file and change the server IP address from 127.0.0.1 to the master node IP address. You will use this copy of the kubeconfig file to access k3s from your computer.

```shell
sudo cp /etc/rancher/k3s/k3s.yaml ~/kubeconfig
```

Copy the edited file to your computer. Export the KUBECONFIG variable from the shell or export it from your ~/.zshrc or ~/.bashrc file.

```shell
scp pi@pi1:~/kubeconfig ~/kubeconfig
export KUBECONFIG=~/kubeconfig
```

Verify that your laptop or desktop computer is configured correctly by getting a list of the nodes:

```shell
$ kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
pi1    Ready    master   3m15s   v1.19.3+k3s3
```

## Installing the Agent Nodes

List the server token from the master node, copy it, and save it for later.

```shell
ssh pi@pi1
sudo cat /var/lib/rancher/k3s/server/node-token
```

Pass the server token into the agent node command to install the k3s agent on the remaining Raspberry Pis in your cluster. You can do this on the nodes individually or use the tip below to install the agent on all of the nodes simultaneously.

{% include tip.html content="Would you like to be able to update all of Raspberry Pis at simultaneously? If so, see my side-note <a href='/how-to-multicast-commands'>How to Multicast Commands</a>. " %}

Whether you update the nodes one at a time or simultaneously, export the server token copied above as a variable, and then run the command shown.

```shell
ssh pi2
export TOKEN=***REDACTED***
curl -sfL https://get.k3s.io agent | INSTALL_K3S_EXEC="agent --server https://192.168.1.10:6443 --token $TOKEN" sh -
```

Wait a couple of minutes while the agent starts-up and registers with the main node. Re-run “kubectl get nodes.” You should see results like those shown below.

{% include image.html url="/img/post-assets/sidenode-k8s-on-raspbian/status_of_nodes.png" description="Status of Kubernetes Nodes" %}

----
## References
* <a href="https://rancher.com/docs/k3s/latest/en/quick-start/">Rancher Docs: Quick-Start Guide</a>
* <a href="https://itnext.io/building-a-kubernetes-cluster-on-raspberry-pi-and-low-end-equipment-part-1-a768359fbba3">Building a kubernetes cluster on Raspberry Pi and low-end equipment. Part 1</a>
* <a href="https://mjpitz.com/blog/2019/04/10/k8s-k3s-rpi-oh-my/">k3s on Raspberry Pi</a>
* <a href="https://medium.com/@marcovillarreal_40011/cheap-and-local-kubernetes-playground-with-k3s-helm-5a0e2a110de9">Cheap and local Kubernetes playground with K3s & Helm</a>
* <a href="https://blog.alexellis.io/test-drive-k3s-on-raspberry-pi/">Will it cluster? k3s on your Raspberry Pi</a>
* <a href="https://itnext.io/building-a-kubernetes-cluster-on-raspberry-pi-and-low-end-equipment-part-1-a768359fbba3">Building a kubernetes cluster on Raspberry Pi and low-end equipment. Part 1</a>
* <a href="https://sysadmins.co.za/running-k3s-on-the-raspberrypi-4/">Kubernetes (K3S) on a RaspberryPi</a>
