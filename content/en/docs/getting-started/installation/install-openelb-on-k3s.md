---
title: "Install OpenELB on K3s"
linkTitle: "Install OpenELB on K3s"
weight: 3
---

This document describes how to use kubectl and [Helm](https://helm.sh/) to install and delete OpenELB in a [K3s](https://k3s.io/) Kubernetes cluster.

{{< notice note >}}

- In a Kubernetes cluster, you only need to install OpenELB once. After the installation is complete, a openelb-manager Deployment that contains a openelb-manager Pod is installed in the cluster. The openelb-manager Pod implements the functionality of OpenELB for the entire Kubernetes cluster.
- After the installation is complete, you can scale the openelb-manager Deployment and assign multiple OpenELB replicas (openelb-manager Pods) to multiple cluster nodes to ensure high availability. For details, see [Configure Multiple OpenELB Replicas](/docs/getting-started/configuration/configure-multiple-openelb-replicas).

{{</ notice >}}

## Prerequisites

* You need to [prepare a K3s Kubernetes cluster](https://rancher.com/docs/k3s/latest/en/installation/), and ensure that the Kubernetes version is 1.15 or later. OpenELB requires CustomResourceDefinition (CRD) v1, which is only supported by Kubernetes 1.15 or later. OpenELB is designed to be used in bare-metal Kubernetes environments. However, you can also use a cloud-based K3s Kubernetes cluster for learning and testing.

* If you use Helm to install OpenELB, ensure that the Helm version is Helm 3.

## Install OpenELB Using kubectl

1. Log in to the master node of the K3s Kubernetes cluster over SSH and run the following command:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml
   ```
   
2. Run the following command to edit the `openelb-manager` Deployment:

   ```bash
   kubectl edit deployment openelb-manager -n openelb-system
   ```

3. Change port 443 to a different value (for example, 30443) to avoid a port conflict with K3s:

   ```yaml
   spec:
     template:
       spec:
         containers:
         - args:
           - --webhook-port=443 # Change the port number.
           ports:
           - containerPort: 443 # Change the port number.
             hostPort: 443 # Change the port number.
   ```

   {{< notice note >}}

   By default, both OpenELB and K3s use port 443. Therefore, you need to change port 443 of OpenELB to avoid the port conflict. For details, see the [official K3s document](https://rancher.com/docs/k3s/latest/en/networking/#traefik-ingress-controller).

   {{</ notice >}}  
   

4. Run the following command to check whether the status of `openelb-manager` is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.

   ```bash
   kubectl get po -n openelb-system
   ```

   It should return something like the following.
   
   ```bash
   NAME                               READY   STATUS      RESTARTS   AGE
   openelb-admission-create-tjsqm     0/1     Completed   0          41s
   openelb-admission-patch-n247f      0/1     Completed   0          41s
   openelb-manager-74c5467674-bx2wg   1/1     Running     0          41s
   ```

## Delete OpenELB Using kubectl

1. To delete OpenELB, log in to the master node of the K3s Kubernetes cluster and run the following command:

   ```bash
   kubectl delete -f https://raw.githubusercontent.com/openelb/openelb/master/deploy/openelb.yaml
   ```

   {{< notice note >}}

   Before deleting OpenELB, you must first delete all Services that use OpenELB.

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
   helm repo add test https://charts.kubesphere.io/test
   helm repo update
   helm install openelb test/openelb
   ```

3. Run the following command to edit the `openelb-manager` Deployment:

   ```bash
   kubectl edit deployment openelb-manager
   ```

4. Change port 443 to a different value (for example, 30443) to avoid a port conflict with K3s:

   ```bash
   spec:
     template:
       spec:
         containers:
         - args:
           - --webhook-port=443 # Change the port number.
           ports:
           - containerPort: 443 # Change the port number.
             hostPort: 443 # Change the port number.
   ```

   {{< notice note >}}

   By default, both OpenELB and K3s use port 443. Therefore, you need to change port 443 of OpenELB to avoid the port conflict. For details, see the [official K3s document](https://rancher.com/docs/k3s/latest/en/networking/#traefik-ingress-controller).

   {{</ notice >}}

5. Run the following command to check whether the status of `openelb-manager` is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.

   ```bash
   kubectl get po -A
   ```

   It should return something like the following.
   
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
   helm delete openelb
   ```

   {{< notice note >}}

   Before deleting OpenELB, you must first delete all Services that use OpenELB.

   {{</ notice >}}

2. Run the following command to check the result. If the OpenELB application does not exist, OpenELB has been deleted successfully.

   ```bash
   helm ls
   ```
