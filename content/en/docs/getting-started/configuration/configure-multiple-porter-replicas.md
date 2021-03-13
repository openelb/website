---
title: "Configure Multiple PorterLB Replicas"
linkTitle: "Configure Multiple PorterLB Replicas"
weight: 4
---

This document describes how to configure multiple PorterLB replicas to ensure high availability in a production environment. You can skip this document if PorterLB is used in a test environment. By default, only one PorterLB replica is installed in a Kubernetes cluster.

* If all Kubernetes cluster nodes are deployed under the same router (BGP mode or Layer 2 mode), you are advised to configure at least two PorterLB replicas, which are installed on two Kubernetes cluster nodes respectively.
* If the Kubernetes cluster nodes are deployed under different leaf routers (BGP mode only), you are advised to configure at least two PorterLB replicas (one replica for one node) under each leaf router. For details, see [Configure PorterLB for Multi-router Clusters](/docs/getting-started/configuration/configure-porter-for-multi-router-clusters/).

## Prerequisites

You need to [prepare a Kubernetes cluster where PorterLB has been installed](/docs/getting-started/installation/).

## Procedure

{{< notice note >}}

The node names and namespace in the following steps are examples only. You need to use the actual values in your environment.

{{</ notice >}}

1. Log in to the Kubernetes cluster and run the following command to label the Kubernetes cluster nodes where PorterLB is to be installed:

   ```bash
   kubectl label --overwrite nodes master1 worker-p002 lb.kubesphere.io/v1alpha1=porter
   ```

   {{< notice note >}}

   In this example, PorterLB will be installed on master1 and worker-p002.

   {{</ notice >}}

2. Run the following command to scale the number of porter-manager Pods to 0:

   ```bash
   kubectl scale deployment porter-manager --replicas=0 -n porter-system
   ```

3. Run the following command to edit the porter-manager Deployment:

   ```bash
   kubectl edit deployment porter-manager -n porter-system
   ```

4. In the porter-manager Deployment YAML configuration, add the following fields under `spec:template:spec`:

   ```yaml
   nodeSelector:
     kubernetes.io/os: linux
     lb.kubesphere.io/v1alpha1: porter
   ```

5. Run the following command to scale the number of porter-manager Pods to the required number (change the number `2` to the actual value):

   ```bash
   kubectl scale deployment porter-manager --replicas=2 -n porter-system
   ```

6. Run the following command to check whether PorterLB has been installed on the required nodes.

   ```bash
   kubectl get po -n porter-system -o wide
   ```

   ![verify-configuration-result](/images/docs/getting-started/configuration/configure-multiple-porter-replicas/verify-configuration-result.jpg)

{{< notice note >}}

* In Layer 2 mode, PorterLB uses the leader election feature of Kubernetes to ensure that only one replica responds to ARP/NDP requests. 
* In BGP mode, all PorterLB replicas will respond to the BgpPeer configuration and attempt to establish a BGP connection with the peer BGP router by default. If the Kubernetes cluster nodes are deployed under different routers, you need to perform further configuration so that the PorterLB replicas establish BGP connections with the correct BGP routers. For details, see [Configure PorterLB for Multi-router Clusters](/docs/getting-started/configuration/configure-porter-for-multi-router-clusters/).

{{</ notice >}}