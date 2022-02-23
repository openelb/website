---
title: "Configure Multiple OpenELB Replicas"
linkTitle: "Configure Multiple OpenELB Replicas"
weight: 4
---

This document describes how to configure multiple OpenELB replicas to ensure high availability in a production environment. You can skip this document if OpenELB is used in a test environment. By default, only one OpenELB replica is installed in a Kubernetes cluster.

* If all Kubernetes cluster nodes are deployed under the same router (BGP mode or Layer 2 mode), you are advised to configure at least two OpenELB replicas, which are installed on two Kubernetes cluster nodes respectively.
* If the Kubernetes cluster nodes are deployed under different leaf routers (BGP mode only), you are advised to configure at least two OpenELB replicas (one replica for one node) under each leaf router. For details, see [Configure OpenELB for Multi-router Clusters](/docs/getting-started/configuration/configure-openelb-for-multi-router-clusters/).

## Prerequisites

You need to [prepare a Kubernetes cluster where OpenELB has been installed](/docs/getting-started/installation/).

## Procedure

{{< notice note >}}

The node names and namespace in the following steps are examples only. You need to use the actual values in your environment.

{{</ notice >}}

1. Log in to the Kubernetes cluster and run the following command to label the Kubernetes cluster nodes where OpenELB is to be installed:

   ```bash
   kubectl label --overwrite nodes master1 worker-p002 lb.kubesphere.io/v1alpha1=openelb
   ```

   {{< notice note >}}

   In this example, OpenELB will be installed on master1 and worker-p002.

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

{{< notice note >}}

* In Layer 2 mode, OpenELB uses the leader election feature of Kubernetes to ensure that only one replica responds to ARP/NDP requests. 
* In BGP mode, all OpenELB replicas will respond to the BgpPeer configuration and attempt to establish a BGP connection with the peer BGP router by default. If the Kubernetes cluster nodes are deployed under different routers, you need to perform further configuration so that the OpenELB replicas establish BGP connections with the correct BGP routers. For details, see [Configure OpenELB for Multi-router Clusters](/docs/getting-started/configuration/configure-openelb-for-multi-router-clusters/).

{{</ notice >}}