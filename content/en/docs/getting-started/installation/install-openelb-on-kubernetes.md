---
title: "Install OpenELB on Kubernetes"
linkTitle: "Install OpenELB on Kubernetes"
weight: 1
---

This document describes how to use kubectl and [Helm](https://helm.sh/) to install and delete OpenELB in a Kubernetes cluster. 

{{< notice note >}}

- In a Kubernetes cluster, you only need to install OpenELB once. After the installation is complete, a openelb-manager Deployment that contains a openelb-manager Pod is installed in the cluster. The openelb-manager Pod implements the functionality of OpenELB for the entire Kubernetes cluster.
- After the installation is complete, you can scale the openelb-manager Deployment and assign multiple OpenELB replicas (openelb-manager Pods) to multiple cluster nodes to ensure high availability. For details, see [Configure Multiple OpenELB Replicas](/docs/getting-started/configuration/configure-multiple-openelb-replicas).

{{</ notice >}}

## Prerequisites

* You need to prepare a Kubernetes cluster, and ensure that the Kubernetes version is 1.15 or later. OpenELB requires CustomResourceDefinition (CRD) v1, which is only supported by Kubernetes 1.15 or later. You can use the following methods to deploy a Kubernetes cluster:

  * Use [KubeKey](https://kubesphere.io/docs/installing-on-linux/) (recommended). You can use KubeKey to deploy a Kubernetes cluster with or without KubeSphere.
  * Follow [official Kubernetes guides](https://kubernetes.io/docs/home/).

  OpenELB is designed to be used in bare-metal Kubernetes environments. However, you can also use a cloud-based Kubernetes cluster for learning and testing.

  * If you use Helm to install OpenELB, ensure that the Helm version is Helm 3.

## Install OpenELB Using kubectl

1. Log in to the Kubernetes cluster over SSH and run the following command:

   We recommend using the stable release version in a production environment. Please use the following command to download the installation script for the stable version:
   
   ```bash
   wget https://raw.githubusercontent.com/openelb/openelb/release-0.5/deploy/openelb.yaml
   kubectl apply -f openelb.yaml
   ```
   
   If you are interested in the latest features of the master branch and want to use the development version, use the following command to download the deployment script:
   
   ```bash
   wget https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml
   kubectl apply -f openelb.yaml
   ```
   
2. Run the following command to check whether the status of `openelb-manager` is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.

   ```bash
   kubectl get po -n openelb-system
   ```

   The following is an example of the expected command output:

   ```bash
   NAME                               READY   STATUS      RESTARTS   AGE
   openelb-admission-create-tjsqm     0/1     Completed   0          41s
   openelb-admission-patch-n247f      0/1     Completed   0          41s
   openelb-manager-74c5467674-bx2wg   1/1     Running     0          41s
   ```

## Delete OpenELB Using kubectl

1. To delete OpenELB, log in to the Kubernetes cluster and run the following command:

   ```bash
   kubectl delete -f openelb.yaml
   ```

   {{< notice note >}}

   Before deleting OpenELB, you must first delete all Services that use OpenELB.

   {{</ notice >}}

2. Run the following command to check the result. If the `openelb-system` namespace does not exist, OpenELB has been deleted successfully.

   ```bash
   kubectl get ns
   ```
   

## Install OpenELB Using Helm

1. Log in to the Kubernetes cluster over SSH and run the following commands:

   ```bash 
   helm repo add kubesphere-stable https://charts.kubesphere.io/stable
   helm repo update
   kubectl create ns openelb-system
   helm install openelb kubesphere-stable/openelb -n openelb-system
   ```

2. Run the following command to check whether the status of `openelb-manager` is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.

   ```bash
   kubectl get po -n openelb-system
   ```

   The following is an example of the expected command output:
   
   ```bash
   NAMESPACE        NAME                              READY   STATUS      RESTARTS   AGE
   openelb-system   openelb-admission-create-m2p52    0/1     Completed   0          32s
   openelb-system   openelb-admission-patch-qmvnq     0/1     Completed   0          31s
   openelb-system   openelb-manager-74c5467674-pgtmh  1/1     Running     0          32s
   ... ...
   ```

## Delete OpenELB Using Helm

1. To delete OpenELB, run the following command:

   ```bash
   helm delete openelb -n openelb-system
   ```

   {{< notice note >}}

   Before deleting OpenELB, you must first delete all Services that use OpenELB.

   {{</ notice >}}

2. Run the following command to check the result. If the OpenELB application does not exist, OpenELB has been deleted successfully.

   ```bash
   helm ls -n openelb-system
   ```
