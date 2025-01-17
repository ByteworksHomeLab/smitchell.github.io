---
layout: post
title:  "How NOT to Home Lab"
url: /how-not-tp-home-lab
comments: true
date: 2021-11-03 17:20:08
categories: kubernetes
author_name : Steve Mitchell
author_url : /author/steve
author_avatar: steve
show_avatar : false
read_time : 10
feature_image: feature-kubernetes
show_related_posts: false
square_related: feature-kubernetes
---
<a href="./how-NOT-to-home-lab">
    <img 
        src="/img/post-assets/2021-11-03-how-not-to-home-lab/HomeLab.png" 
        alt="Bare Metal Home Lab - circa 2020"
    >
</a>

I learned how not to build a home lab, at least in my case. I decided to avoid small machines and go with big, multiprocessor machines with a fast hypervisor (type-1), that is, because I wasn't enjoying tinkering with my Raspberry Pis anymore.

In 2020, I built a bare-metal home lab consisting of a Raspberry Pi K3S Kubernetes cluster and a cluster of Mac Minis. Running Kubernetes on Raspberry Pis was fun, but eventually, I needed more clusters running different Kubernetes distributions. The problem with bare metal is that it is inflexible and can get expensive over time. It was time to re-think my home lab strategy.

There are lots of ways to “home lab” when it comes to Kubernetes. The quickest way to get started is using your desktop/laptop and any of these tools:

* <a href="https://birthday.play-with-docker.com/kubernetes-docker-desktop/">Docker Engine</a> - Docker for the desktop with built-in Kubernetes.
* <a href="https://k3s.io">K3S</a> - Lightweight Rancher distribution.
* <a href="https://kind.sigs.k8s.io">Kind</a> (Kubernetes in Docker)
* <a href="https://microk8s.io">Micro8s</a> - Lightway Canonical distribution.
* <a href="https://minikube.sigs.k8s.io/docs/start/">Minikube</a> - A community version of Kubernetes for learning. 
* <a href="https://github.com/minishift/minishift">MiniShift</a> - A fork of MiniKube for running OpenShift.
* <a href="https://rancherdesktop.io">Rancher Desktop</a> - Rancher Desktop is an alternative to Docker Engine that runs K3S.
* <a href="https://tanzucommunityedition.io">Tanzu Community Edition</a> - Tanzu Kubernetes Engine that runs on Kind.
* For virtualization there is Vagrant, VirtualBox, VMware Fusion, VMware Workstation, and more.

Desktop containerization/virtualization is the simplest, least costly option for learning at home. You can stop reading here if some combination of these desktop tools satisfies your home lab needs.

What if you need more? Do you go Cloud or on-premises? That depends on your requirements.
## Cloud
If you want to spin up short-lived clusters in the Cloud to avoid costs, I recommend using Terraform and Ansible for IaC (Infrastructure as Code). IoC lets you create and destroy Kubernetes clusters on a whim. As an application developer, I don’t have to use Terraform and Ansible much at work, but I use them in my lab.
## On-premises
If you need long-lived K8S clusters, for instance, to run <a href="https://www.youtube.com/watch?v=icyTnoonRqI">home automation with Home Assistant</a>, you should find a multi-core, high-memory machine to do virtualization. I bought a used, dual-Xeon machine with 64 GB of RAM for cheap. 

{% include tip.html content="When I re-imaged that 8-year-old machine, Centos, and later, vSphere EXSi warned that support for its firmware may end in an upcoming release. Maybe you should spend a little more money for a little less vintage machine. 😊" %}

## Hypervisors
There are two types of hypervisor to do virtualization: type-1 or type-2.
### Type-1 Hypervisors
Type-1 hypervisors run directly on bare metal and offer the best performance. Examples of type-1 hypervisors are VMware ESXi, Red Hat Virtualization Server, Microsoft Hyper V, and Citrix Xen. Open source Xen was a popular type-1 hypervisor in the early days of virtualization. KMS has largely supplanted Xen. Other than open-source Xen, the rest are commercial products, so you’ll need to determine the product licensing. For example, if you buy the <a href="https://www.vmug.com/membership/vmug-advantage-membership/">VMware User Group Advantage</a> membership for $200/year, you can access all the VMware enterprise products (with use restrictions).

The illustration below shows that with type-1, the hypervisor and OS are the same. 

{% include image.html url="/img/post-assets/2021-11-03-how-not-to-home-lab/Type 1 Hypervisor.png" description="Type-1 Hypervisor" %}
## Type-2 Hypervisors
There are many open-source type-2 hypervisors available for your home lab, but they are slower since they run on top of the OS.  You typically don’t see companies run VirtualBox, a type-2 hypervisor, in production because of its slow performance, but it is well suited for home lab applications. 

Other type-2 hypervisors include VMware Workstation/Fusion/Player and Microsoft Virtual PC. VirtualBox seems to be the most popular, with lots of prebuilt images available online.

{% include image.html url="/img/post-assets/2021-11-03-how-not-to-home-lab/Type 2 Hypervisor.png" description="Type-2 Hyperviso.png"%}
## KVM Hypervisor
Then there is the KVM hypervisor, which is a mix of type-1 and type-2 hypervisors. KVM is nearly as fast as a type-1 hypervisor because it is built into the Linux kernel. <a href="https://www.proxmox.com/en/">Proxmox</a>, <a href="https://virt-manager.org">Virt-Manager</a>, and <a href="https://cockpit-project.org">Cockpit</a> are GUIs you can run on top of KVM.

As you can see in the illustration below, the guest OSs have direct access to the KVM hypervisor. The overhead is as little as 5%. It also has an extensive CLI, so it is well suited for automation with tools like Terraform.
{% include image.html url="/img/post-assets/2021-11-03-how-not-to-home-lab/KVM Architecture.png" description="KVM Hypervisor" %}
## Choosing an On-premises Hypervisor
If the purpose of your lab is to help you in your current job, and you work for a company that runs VMware, Microsoft, Citrix, or Red Hat virtualization products, you should build something that mirrors your work environment. 

You will have to deal with licensing, though. As I mentioned, if you buy the VMUG Advantage membership, you get access to all the VMware products for $200/year. Please see the <a href="https://www.vmware.com/resources/compatibility/search.php">VMware compatibility matrix</a> to find the proper hardware.

If you don’t need a commercial hypervisor for work, you should use Proxmox or KVM. Proxmox has a built-in GUI, or if you go with native KVM, and don’t want to use the KVM CLI, you can add the <a href="https://virt-manager.org">Virt-Manager</a> or <a href="https://cockpit-project.org">Cockpit</a> GUI on top of KVM.

{% include tip.html content="Robert Sirchia (<a href='https://twitter.com/robertsirc'>@robertsirc</a>), Senior Technical Evangelist at SUSE, made a suggestion for this blog. Robert recommends you give <a href='https://community.suse.com/posts/harvester-intro-and-setup'>Harvester</a> a try." %}

According to the <a href="https://docs.harvesterhci.io">Harvester site</a>, "Harvester is an open-source hyper-converged infrastructure (HCI) software built on Kubernetes. It is an open alternative to using a proprietary HCI stack that incorporates the design and ethos of Cloud Native Computing."

Harvester looks really nice. I will definitely give it a try.

{% include image.html url="/img/post-assets/2021-11-03-how-not-to-home-lab/create-vm.png" description="Harvester GUI"%}

I wasn't aware of Harvester when I originally imaged my "new" lab workstation, so I went with Centos and Cockpit. It used a bridged network, so all my physical and virtual machines shared the same subnet. Here is how it looked with the Cockpit GUI. 

{% include image.html url="/img/post-assets/2021-11-03-how-not-to-home-lab/cockpit.png" description="KVM with Cockpit"%}

Many of my employer's customers are VMware customers too, so it is in my best interest to buy the VMUG Advantage membership and install vCenter. That takes a lot more work than KVM, but I hope it is worth it in the long run. The only caveat is that Tanzu on vSphere (specifically HAProxy for Tanzu) requires <a href="https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-C3048E95-6E9D-4AC3-BE96-44446D288A7D.html#GUID-C3048E95-6E9D-4AC3-BE96-44446D288A7D">three ESXi hosts with 8-cores and 64 GB RAM</a>. Yikes!

I'll close with a screenshot of what made me realize that Tanzu is going to jack-up the cost of my home lab. I’ll post an update when Tanzu is up and running.

{% include image.html url="/img/post-assets/2021-11-03-how-not-to-home-lab/haproxy_requirements.png" description="Tanzu HAProxy Requirements"%}
