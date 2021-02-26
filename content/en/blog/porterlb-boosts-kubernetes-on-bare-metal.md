---
title: "PorterLB for Bare-Metal Kubernetes: Cloud Native, Elegant, and Flexible"
linkTitle: "PorterLB for Bare-Metal Kubernetes: Cloud Native, Elegant, and Flexible"
description: Use PorterLB to expose applications in bare-metal Kubernetes clusters.
keywords: PorterLB, Bare Metal, Kubernetes, KubeSphere, Load Balancer, Cloud Native
author: Patrick
date: 2021-02-25
weight: 100000
---

Applications deployed on Kubernetes are usually exposed by using LoadBalancer Services, which rely heavily on load balancer implementations provided by cloud vendors. For applications deployed in bare-metal Kubernetes clusters, where cloud-based load balancer implementations are unavailable, Kubernetes has yet to provide a viable LoadBalancer solution.

PorterLB well addresses this problem. As a sub-project of [KubeSphere](https://kubesphere.io/), PorterLB boosts application containerization in bare-metal Kubernetes environments, and complements the KubeSphere ecosystem in the bare-metal field.

In this article, I am going to introduce how PorterLB functions in Layer 2 mode and BGP mode, which are provided for users to expose applications in different scenarios, and the advantages of PorterLB compared with other bare-metal load balancer implementations such as MetalLB.

## BGP Mode

In BGP mode, PorterLB publishes routes to a BGP router deployed outside the Kubernetes cluster, and the BGP router forwards Service traffic from external clients to the Kubernetes cluster nodes based on the routes obtained from PorterLB. In this process, PorterLB uses Equal-Cost Multi-Path (ECMP) routing to ensure that all Kubernetes nodes or nodes that contain Pods, depending on the user configuration, are used as next hops by the BGP router.

![porter-bgp-topology](/images/blog/porterlb-boosts-kubernetes-on-bare-metal/porter-bgp-topology.jpg)

The process of using PorterLB in BGP mode in a Kubernetes cluster is simple:

1. Install PorterLB.
2. Configure an IP address pool by using Eip.
3. Configure BGP properties by using BgpConf and BgpPeer.
4. Create a Service and set the Service to use PorterLB, which is similar to what you do to use a load balancer plugin in a cloud-based Kubernetes cluster.

PorterLB can be configured by using the Eip, BgpConf, and BgpPeer CRDs, no other configuration files are required. In addition, as BGP is decentralized, you can use the BGP mode to easily establish a high availability network free of failover interruptions and bandwidth bottlenecks.

![high-availability-network](/images/blog/porterlb-boosts-kubernetes-on-bare-metal/high-availability-network.jpg)

## Layer 2 Mode

Generally, you are advised to use the BGP mode to expose your Services in a high availability network. However, BGP may be unavailable in certain systems because of security requirements or because the router does not support BGP. In this case, you can use PorterLB in Layer 2 mode to achieve similar functionality.

In Layer 2 mode, PorterLB uses ARP packets (for IPv4) or NDP packets (for IPv6) to map the Service IP address to the MAC address of a Kubernetes node. The mechanism of the Layer 2 mode is similar to that of the BGP mode, except that BGP is replaced with ARP/NDP and the router obtains only one route destined for the Service. 

![porter-layer-2-topology](/images/blog/porterlb-boosts-kubernetes-on-bare-metal/porter-layer-2-topology.jpg)

Though the Layer 2 mode does not provide the same high availability as the BGP mode does, it is easier to use (you don't even need to configure BGP properties):

1. Install PorterLB.
2. Configure an IP address pool by using Eip.
3. Create a Service and set the Service to use PorterLB.

You can obtain detailed guidance on how to install, configure, and use PorterLB from the [PorterLB documentation](https://porterlb.io/docs/).

## Advantages of PorterLB

There are other load balancer implementations such as MetalLB designed for bare-metal Kubernetes clusters. Compared with others, PorterLB has the following advantages.

### Cloud native

To manage IP address pools and BGP properties for PorterLB, you only need to use the `kubectl apply` command provided by Kubernetes to create CRD objects. To obtain the status information about IP address pools and BGP peers, you can simply run `kubectl get` to view the status of the CRD objects. No other configuration files are required. In addition, a PorterLB GUI will be available soon, which will further simplify the usage of PorterLB.

### Elegant

After PorterLB is installed in a Kubernetes cluster, a porter-manager Deployment that contains a porter-manager Pod is created. The porter-manager Pod implements the functionality of PorterLB for the entire Kubernetes cluster. For high availability, you can scale the porter-manager Deployment and assign multiple PorterLB replicas (porter-manager pods) to multiple cluster nodes. This simple architecture ensures that PorterLB can be easily managed and integrated with other systems.

### Flexible

PorterLB can be used in conventional Kubernetes clusters. As a sub-project of KubeSphere, PorterLB also fits well into the KubeSphere ecosystem. You can seamlessly integrate PorterLB as a plugin with KubeSphere to utilize the abundant features of the KubeSphere ecosystem, such as observability and troubleshooting, unified monitoring and logging, centralized storage and networking management, and easy-to-use CI/CD pipelines.