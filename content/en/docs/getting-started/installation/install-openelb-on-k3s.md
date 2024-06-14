---
title: "Install OpenELB on K3s"
linkTitle: "Install OpenELB on K3s"
weight: 3
---

This document describes how to use kubectl and [Helm](https://helm.sh/) to install and delete OpenELB in a [K3s](https://k3s.io/) Kubernetes cluster.

{{< notice note >}}

- In a Kubernetes cluster, you only need to install OpenELB once. After the installation is complete, an openelb-controller Deployment containing an openelb-controller Pod and an openelb-speaker DaemonSet containing openelb-speaker Pods will be installed in the cluster. The openelb-controller Pod is used to implement the IPAM for service load balancer IPs, while the openelb-speaker Pods are used to announce the service load balancer IPs.

- After the installation is complete, you can set node selectors to ensure that the load traffic runs on specific nodes. For details, see [Configure Multiple OpenELB Replicas](/docs/getting-started/configuration/configure-multiple-openelb-replicas).

{{</ notice >}}

## Prerequisites

* You need to [prepare a K3s Kubernetes cluster](https://rancher.com/docs/k3s/latest/en/installation/), and ensure that the Kubernetes version is 1.15 or later. OpenELB requires CustomResourceDefinition (CRD) v1, which is only supported by Kubernetes 1.15 or later. OpenELB is designed to be used in bare-metal Kubernetes environments. However, you can also use a cloud-based K3s Kubernetes cluster for learning and testing.

* If you use Helm to install OpenELB, ensure that the Helm version is Helm 3.

## Install OpenELB Using kubectl

1. Log in to the master node of the K3s Kubernetes cluster over SSH and run the following command:

   We recommend using the stable release version in a production environment. Please use the following command to download the installation script for the stable version:
   
   ```bash
   wget https://raw.githubusercontent.com/openelb/openelb/release-0.6/deploy/openelb.yaml
   kubectl apply -f openelb.yaml
   ```
   
   If you are interested in the latest features of the master branch and want to use the development version, use the following command to download the deployment script:
   
   ```bash
   wget https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml
   kubectl apply -f openelb.yaml
   ```
   
2. Run the following command to check whether the status of `openelb-controller` and `openelb-speaker` is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.
   ```bash
   kubectl get po -n openelb-system
   ```

   It should return something like the following.

   ```bash
   NAME                                  READY   STATUS      RESTARTS   AGE
   openelb-admission-create-fv8jb        0/1     Completed   0          41s
   openelb-admission-patch-887qh         0/1     Completed   0          41s
   openelb-controller-6d59c894c9-jgfks   1/1     Running     0          41s
   openelb-speaker-dnkcr                 1/1     Running     0          41s
   openelb-speaker-fmkxb                 1/1     Running     0          41s
   openelb-speaker-trh6p                 1/1     Running     0          41s
   ```

## Delete OpenELB Using kubectl

1. To delete OpenELB, log in to the master node of the K3s Kubernetes cluster and run the following command:

   ```bash
   kubectl delete -f openelb.yaml
   ```

   {{< notice note >}}

   Before deleting OpenELB, you should first delete all Services that use OpenELB.

   {{</ notice >}}

2. Run the following command to check the result. If the openelb-system namespace does not exist, OpenELB has been deleted successfully.

   ```bash
   kubectl get ns
   ```
   

## Install OpenELB Using Helm

1. Log in to the master node of the K3s Kubernetes cluster over SSH and run the following command to configure the environment variable:

   ```bash
   export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
   ```

   {{< notice note >}}

   You need to configure this environment variable so that Helm can reach the K3s Kubernetes cluster. For details, see the [official K3s document](https://rancher.com/docs/k3s/latest/en/cluster-access/).

   {{</ notice >}}

2. Run the following commands to install OpenELB:

   ```bash 
   helm repo add openelb https://openelb.github.io/openelb
   helm repo update
   helm install openelb openelb/openelb -n openelb-system --create-namespace
   ```

3. Run the following command to check whether the status of `openelb-controller` and `openelb-speaker` is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.

   ```bash
   kubectl get po -A
   ```

   It should return something like the following.
   
   ```bash
   NAMESPACE            NAME                                  READY   STATUS      RESTARTS   AGE
   openelb-system       openelb-admission-create-fv8jb        0/1     Completed   0          41s
   openelb-system       openelb-admission-patch-887qh         0/1     Completed   0          41s
   openelb-system       openelb-controller-6d59c894c9-jgfks   1/1     Running     0          41s
   openelb-system       openelb-speaker-dnkcr                 1/1     Running     0          41s
   openelb-system       openelb-speaker-fmkxb                 1/1     Running     0          41s
   openelb-system       openelb-speaker-trh6p                 1/1     Running     0          41s
   ... ...
   ```
   
   

## Delete OpenELB Using Helm

1. To delete OpenELB, run the following command:

   ```bash
   helm delete openelb -n openelb-system
   ```

   {{< notice note >}}

   Before deleting OpenELB, you should first delete all Services that use OpenELB.

   {{</ notice >}}

2. Run the following command to check the result. If the OpenELB application does not exist, OpenELB has been deleted successfully.

   ```bash
   helm ls -n openelb-system
   ```
