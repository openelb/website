---
title: "Layer 2 Mode"
linkTitle: "Layer 2 Mode"
weight: 1
---

This document describes the network topology of PorterLB in Layer 2 mode and how PorterLB functions in Layer 2 mode.

{{< notice note >}}

* Generally, you are advised to use the BGP mode because it allows you to create a high availability system free of failover interruptions and bandwidth bottlenecks. However, the BGP mode requires your router to support BGP and Equal-Cost Multi-Path (ECMP) routing, which may be unavailable in certain systems. In this case, you can use the Layer 2 mode to achieve similar functionality.
* The Layer 2 mode requires your infrastructure environment to allow anonymous ARP packets. If PorterLB is installed in a cloud-based Kubernetes cluster for testing, you need to confirm with your cloud vendor whether anonymous ARP packets are allowed. If not, the Layer 2 mode cannot be used.

{{</ notice >}}

## Network Topology

The following figure shows the topology of the network between a Kubernetes cluster with PorterLB and a router.

![porter-layer-2-topology](/images/docs/concepts/layer-2-mode/porter-layer-2-topology.jpg)

IP addresses and MAC addresses in the preceding figure are examples only. The topology is described as follows:

* A Service backed by two Pods is deployed in the Kubernetes cluster, and is assigned an IP address 192.168.0.91 for external access. The Service IP address is on the same network segment as the cluster node IP addresses.
* PorterLB installed in the Kubernetes cluster randomly selects a node (worker 1 in this example) to handle Service requests. After that, PorterLB sends an ARP/NDP packet to the router, which maps the Service IP address to the MAC address of worker 1.
* If multiple porter-manager replicas have been deployed in the cluster, PorterLB uses the the leader election feature of Kubernetes to ensure that only one replica responds to ARP/NDP requests. 
* When an external client machine attempts to access the Service, the router forwards the Service traffic to worker 1 based on the mapping between the Service IP address and the MAC address of worker 1. After the Service traffic reaches worker 1, kube-proxy can further forward the traffic to other nodes for load balancing (both Pod 1 and Pod 2 can be reached over kube-proxy).
* If worker 1 fails, PorterLB re-sends an APR/NDP packet to the router to map the Service IP address to the MAC address of worker 2, and the Service traffic switches to worker 2.

{{< notice note >}}

The Layer 2 mode has two limitations:

* Worker 1 and worker 2 work in active-standby mode in terms of traffic forwarding. When a failover occurs, Services in the Kubernetes cluster will be interrupted for a short while.
* All Service traffic is always sent to one node first and then forwarded to other nodes over kube-proxy in a second hop. Therefore, the Service bandwidth is limited to the bandwidth of a single node, which causes a bandwidth bottleneck.

{{</ notice >}}
