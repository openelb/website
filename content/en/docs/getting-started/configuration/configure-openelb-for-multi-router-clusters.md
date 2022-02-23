---
title: "Configure OpenELB for Multi-router Clusters (BGP Mode)"
linkTitle: "Configure OpenELB for Multi-router Clusters (BGP Mode)"
weight: 3
---

This document describes how to configure OpenELB in BGP mode for Kubernetes cluster nodes deployed under multiple routers. You can skip this document if all Kubernetes cluster nodes are deployed under the same router.

{{< notice note >}}

This document applies only to the BGP mode. The Layer 2 mode requires that all Kubernetes cluster nodes be on the same Layer 2 network (under the same router).

{{</ notice >}}

## Network Topology Before Configuration

This section explains why you need to perform the configuration. The following figure shows the network topology of a Kubernetes cluster before the configuration.

![multi-router-topology-1](/images/en/docs/getting-started/configuration/configure-openelb-for-multi-router-clusters/multi-router-topology-1.jpg)

IP addresses in the preceding figure are examples only. The topology is described as follows:

* In the Kubernetes cluster, the master and worker 1 nodes are deployed under the leaf 1 BGP router, and the worker 2 node is deployed under the leaf 2 BGP router. OpenELB is only installed under leaf 1 (by default, only one OpenELB replica is installed).
* A Service backed by two Pods is deployed in the Kubernetes cluster, and is assigned an IP address 172.22.0.2 for external access. Pod 1 and Pod 2 are deployed on worker 1 and worker 2 respectively.
* OpenELB establishes a BGP connection with leaf 1 and publishes the IP addresses of the master node and worker 1 (192.168.0.3 and 192.168.0.4) to leaf 1 as the next hop destined for the Service IP address 172.22.0.2.
*  Leaf 1 establishes a BGP connection with the spine BGP router and publishes its own IP address 192.168.0.2 to the spine router as the next hop destined for the Service IP address 172.22.0.2.
* When an external client machine attempts to access the Service, the spine router forwards the Service traffic to leaf 1, and leaf 1 load balances the traffic among the master node and worker 1.
* Although Pod 2 on worker 2 can also be reached over kube-proxy, router-level load balancing is implemented only among the master node and worker 1 and the Service bandwidth is limited to the bandwidth of the master node and worker 1.

To resolve the problem, you need to label the Kubernetes cluster nodes and change the OpenELB Deployment configuration so that OpenELB is installed on nodes under all leaf routers. In addition, you need to specify the [spec:nodeSelector:matchLabels](/docs/getting-started/configuration/configure-openelb-in-bgp-mode/#configure-peer-bgp-properties-using-bgppeer) field in the BgpPeer configuration so that the OpenELB replicas establish BGP connections with the correct BGP routers.

## Network Topology After Configuration

This section describes the configuration result you need to achieve. The following figure shows the network topology of a Kubernetes cluster after the configuration.

![multi-router-topology-2](/images/en/docs/getting-started/configuration/configure-openelb-for-multi-router-clusters/multi-router-topology-2.jpg)

IP addresses in the preceding figure are examples only. The topology is described as follows:

* After the configuration, OpenELB is installed on nodes under all leaf routers.
* In addition to [what happens before the configuration](#network-topology-before-configuration), the OpenELB replica installed under leaf 2 also establishes a BGP connection with leaf 2 and publishes the worker 2 IP address 192.168.1.2 to leaf 2 as the next hop destined for the Service IP address 172.22.0.2.
* Leaf 2 establishes a BGP connection with the spine router and publishes its own IP address 192.168.1.1 to the spine router as the next hop destined for the Service IP address 172.22.0.2.
* When an external client machine attempts to access the Service, the spine router load balances the Service traffic among leaf 1 and leaf 2. Leaf 1 load balances the traffic among the master node and worker 1. Leaf 2 forwards the traffic to worker 2. Therefore, the Service traffic is load balanced among all three Kubernetes cluster nodes, and the Service bandwidth of all three nodes can be utilized.

## Configuration Procedure

### Prerequisites

You need to [prepare a Kubernetes cluster where OpenELB has been installed](/docs/getting-started/installation/).

### Procedure

{{< notice note >}}

The node names, leaf router names, and namespace in the following steps are examples only. You need to use the actual values in your environment.

{{</ notice >}}

1. Log in to the Kubernetes cluster and run the following commands to label the Kubernetes cluster nodes where OpenELB is to be installed:

   ```bash
   kubectl label --overwrite nodes master1 worker-p002 lb.kubesphere.io/v1alpha1=openelb
   ```

   {{< notice note >}}

   OpenELB works properly if it is installed on only one node under each leaf router. In this example, OpenELB will be installed on master1 under leaf1 and worker-p002 under leaf2. However, to ensure high availability in a production environment, you are advised to installed OpenELB on at least two nodes under each leaf router.

   {{</ notice >}}

2. Run the following command to scale the number of openelb-manager Pods to 0:

   ```bash
   kubectl scale deployment openelb-manager --replicas=0 -n openelb-system
   ```

3. Run the following command to edit the openelb-manager Deployment:

   ```bash
   kubectl edit deployment openelb-manager -n openelb-system
   ```

4. In the openelb-manager Deployment YAML configuration, add the following fields under `spec:template:spec`:

   ```yaml
   nodeSelector:
     kubernetes.io/os: linux
     lb.kubesphere.io/v1alpha1: openelb
   ```
   
5. Run the following command to scale the number of openelb-manager Pods to the required number (change the number `2` to the actual value):

   ```bash
   kubectl scale deployment openelb-manager --replicas=2 -n openelb-system
   ```

6. Run the following command to check whether OpenELB has been installed on the required nodes.

   ```bash
   kubectl get po -n openelb-system -o wide
   ```

      It should return something like the following.

      ```bash
      NAME                               READY   STATUS      RESTARTS   AGE   IP              NODE    		NOMINATED NODE   READINESS GATES
      openelb-admission-create-m2p52     0/1     Completed   0          49m   10.233.92.34    worker-p001		<none>           <none>
      openelb-admission-patch-qmvnq      0/1     Completed   0          49m   10.233.96.15    worker-p002		<none>           <none>
      openelb-manager-74c5467674-pgtmh   1/1     Running     0          19m   192.168.0.2   	master1   		<none>           <none>
      openelb-manager-74c5467674-wmh5t   1/1     Running     0          19m   192.168.0.4   	worker-p002 	<none>           <none>
      ```

7. Run the following commands to label the Kubernetes cluster nodes so that the OpenELB replicas establish BGP connections with the correct BGP routers.

   ```bash
   kubectl label --overwrite nodes master1 openelb.kubesphere.io/rack=leaf1
   ```

   ```bash
   kubectl label --overwrite nodes worker-p002 openelb.kubesphere.io/rack=leaf2
   ```

8. When creating BgpPeer objects, configure the [spec:nodeSelector:matchLabels](/docs/getting-started/configuration/configure-openelb-in-bgp-mode/#configure-peer-bgp-properties-using-bgppeer) field in the BgpPeer YAML configuration for each leaf router. The following YAML configurations specify that the OpenELB replica on master1 communicates with leaf1, and the OpenELB replica on worker-p002 communicates with leaf2. 

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
   
   




