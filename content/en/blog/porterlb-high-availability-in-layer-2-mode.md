---
title: "PorterLB: High Availability Mechanisms in Layer 2 Mode"
linkTitle: "PorterLB: High Availability Mechanisms in Layer 2 Mode"
description: How PorterLB ensures high availability in Layer 2 mode.
keywords: PorterLB, Bare Metal, Kubernetes, High Availability, Layer 2 
author: Patrick
date: 2021-03-30
weight: 99998
---

PorterLB makes it possible for users to expose Services in bare-metal Kubernetes environments. Currently, PorterLB supports the BGP mode and the Layer 2 mode, which use BGP and ARP/NDP respectively to expose Services.

Generally, the BGP mode is recommended because it allows you to build a high availability system free of failover interruptions and bandwidth bottlenecks. However, BGP may be unavailable in certain systems because of security requirements or because the router does not support BGP. In this case, you can use PorterLB in Layer 2 mode to achieve similar functionality.

Though the Layer 2 mode does not provide the same high availability as the BGP mode, it does implement certain mechanisms to ensure that PorterLB can still function as long as the Kubernetes cluster is not entirely down.

In this article, I am going to discuss the high availability mechanisms of PorterLB in Layer 2 mode. The following scenarios will be examined:

* Scenario 1: Only one PorterLB replica is deployed and the node selected by PorterLB is down.
* Scenario 2: Only one PorterLB replica is deployed and the node where PorterLB is deployed is down.
* Scenario 3: Multiple PorterLB replicas are deployed and one of the nodes that contain PorterLB replicas is down.

## Scenario 1

<img src="/images/blog/porterlb-high-availability-mechanisms-in-layer-2-mode/scenario-1-1.png" width="800px">

In the Kubernetest cluster, only one PorterLB replica (porter-manager Pod) is deployed on node 3 and an application Pod is deployed on node 2. The application Pod is exposed by using a Service (192.168.0.91). PorterLB maps the IP address of the Service to the MAC address of node 2.

If the node (node 2 in this example) selected by PorterLB is down, PorterLB automatically maps the Service IP address to the MAC address of another node to rebuild connection to the Service.

Therefore, the network topology after node 2 is down probably looks like the following:

<img src="/images/blog/porterlb-high-availability-mechanisms-in-layer-2-mode/scenario-1-2.png" width="800px">

One thing you should be aware of is that although PorterLB automatically rebuilds the connection to the Service, there is a short period of failover interruption, which is one of the reasons why the BGP mode better suits scenarios where availability is vital.

## Scenario 2

<img src="/images/blog/porterlb-high-availability-mechanisms-in-layer-2-mode/scenario-2-1.png" width="800px">

So what if the node where the porter-manager Pod is deployed is down?

Well, the porter-manager Pod is deployed under a ReplicaSet in a Deployment. Therefore, if the node where the porter-manager Pod is deployed is down, the Kubernetes system automatically re-creates the porter-manager Pod on another node. The network topology changes to the following:

<img src="/images/blog/porterlb-high-availability-mechanisms-in-layer-2-mode/scenario-2-2.png" width="800px">

Though existing Services that use PorterLB are not affected, the functionality of PorterLB is unavailable during the re-creation, which is why you are advised to deploy multiple PorterLB replicas (porter-manager Pods).

## Scenario 3

<img src="/images/blog/porterlb-high-availability-mechanisms-in-layer-2-mode/scenario-3-1.png" width="800px">

When multiple PorterLB replicas (porter-manager Pods) are deployed, PorterLB uses the leader election mechanism to ensure that only one replica (the leader) communicates with the router. If the node where the leader is located is down, another PorterLB replica automatically takes over the service after a leader re-election. The network topology changes to the following:

<img src="/images/blog/porterlb-high-availability-mechanisms-in-layer-2-mode/scenario-3-2.png" width="800px">

Although the functionality of PorterLB is still unavailable during the leader re-election, the interruption period is much shorter than that in scenario 2. Therefore, if you need to use the Layer 2 mode in a production environment, it is highly recommended that you deploy multiple PorterLB replicas to improve the availability.