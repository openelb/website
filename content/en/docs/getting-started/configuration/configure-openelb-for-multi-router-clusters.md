---
title: "Configure OpenELB for Multi-router Clusters (BGP Mode)"
linkTitle: "Configure OpenELB for Multi-router Clusters (BGP Mode)"
weight: 3
---

This document describes how to configure OpenELB in BGP mode for Kubernetes cluster nodes deployed under multiple routers.

{{< notice note >}}

This document applies only to the BGP mode. The Layer 2 or VIP mode requires that all Kubernetes cluster nodes be on the same Layer 2 network (under the same router).

{{</ notice >}}

## Network Topology

This section describes the configuration result you need to achieve. The following figure shows the network topology of a Kubernetes cluster after the configuration.

![multi-router-topology-2](/images/en/docs/getting-started/configuration/configure-openelb-for-multi-router-clusters/multi-router-topology-2.png)

IP addresses in the preceding figure are examples only. The topology is described as follows:

* In the Kubernetes cluster, the master and worker 1 nodes are deployed under the leaf 1 BGP router, and the worker 2 node is deployed under the leaf 2 BGP router. OpenELB is installed on nodes under all leaf routers.
* A Service backed by two Pods is deployed in the Kubernetes cluster, and is assigned an IP address 172.22.0.2 for external access. Pod 1 and Pod 2 are deployed on worker 1 and worker 2 respectively.
* The OpenELB replica installed under leaf1 establishes a BGP connection with leaf 1 and publishes the IP addresses of the master node and worker 1 (192.168.0.3 and 192.168.0.4) to leaf 1 as the next hop destined for the Service IP address 172.22.0.2.
* The OpenELB replica installed under leaf 2 establishes a BGP connection with leaf 2 and publishes the worker 2 IP address 192.168.1.2 to leaf 2 as the next hop destined for the Service IP address 172.22.0.2.
* Leaf 1 establishes a BGP connection with the spine BGP router and publishes its own IP address 192.168.0.2 to the spine router as the next hop destined for the Service IP address 172.22.0.2.
* Leaf 2 establishes a BGP connection with the spine router and publishes its own IP address 192.168.1.1 to the spine router as the next hop destined for the Service IP address 172.22.0.2.
* When an external client machine attempts to access the Service, the spine router load balances the Service traffic among leaf 1 and leaf 2. Leaf 1 load balances the traffic among the master node and worker 1. Leaf 2 forwards the traffic to worker 2. Therefore, the Service traffic is load balanced among all three Kubernetes cluster nodes, and the Service bandwidth of all three nodes can be utilized.


## Configuration Procedure

### Prerequisites

You need to [prepare a Kubernetes cluster where OpenELB has been installed](/docs/getting-started/installation/).

### Procedure

{{< notice note >}}

The node names, leaf router names, and namespace in the following steps are examples only. You need to use the actual values in your environment.

{{</ notice >}}

1. Log in to the Kubernetes cluster and run the following commands to label the Kubernetes cluster nodes where openelb-speaker is to be scheduled:

   ```bash
   kubectl label --overwrite nodes master1 worker-p002 lb.kubesphere.io/v1alpha1=openelb
   ```

   {{< notice note >}}

   OpenELB works properly if it is installed on only one node under each leaf router. In this example, OpenELB will be installed on master1 under leaf1 and worker-p002 under leaf2. However, to ensure high availability in a production environment, you are advised to installed OpenELB on at least two nodes under each leaf router.

   {{</ notice >}}

2. Run the following command to edit the openelb-speaker DaemonSet:

   ```bash
   kubectl edit ds openelb-speaker -n openelb-system
   ```

3. In the openelb-speaker DaemonSet YAML configuration, add the following fields under `spec:template:spec`:

   ```yaml
   nodeSelector:
     kubernetes.io/os: linux
     lb.kubesphere.io/v1alpha1: openelb
   ```
 
4. Run the following commands to label the Kubernetes cluster nodes so that the openelb-speaker instances establish BGP connections with the correct BGP routers.

   ```bash
   kubectl label --overwrite nodes master1 openelb.kubesphere.io/rack=leaf1
   kubectl label --overwrite nodes worker-p002 openelb.kubesphere.io/rack=leaf2
   ```

5. When creating BgpPeer objects, configure the [spec:nodeSelector:matchLabels](/docs/getting-started/configuration/configure-openelb-in-bgp-mode/#configure-peer-bgp-properties-using-bgppeer) field in the BgpPeer YAML configuration for each leaf router. The following YAML configurations specify that the OpenELB replica on master1 communicates with leaf1, and the OpenELB replica on worker-p002 communicates with leaf2. 

   ```yaml
   # BgpPeer YAML for master1 and leaf1
   nodeSelector:
       matchLabels:
         openelb.kubesphere.io/rack: leaf1
   ```
   
   ```yaml
   # BgpPeer YAML for worker-p002 and leaf2
   nodeSelector:
       matchLabels:
         openelb.kubesphere.io/rack: leaf2
   ```
   





