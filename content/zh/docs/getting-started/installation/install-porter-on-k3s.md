---
title: "Install PorterLB on K3s"
linkTitle: "Install PorterLB on K3s"
weight: 3
---

This document describes how to use kubectl and [Helm](https://helm.sh/) to install and delete PorterLB in a [K3s](https://k3s.io/) Kubernetes cluster.

{{< notice note >}}

- In a Kubernetes cluster, you only need to install PorterLB once. After the installation is complete, a porter-manager Deployment that contains a porter-manager Pod is installed in the cluster. The porter-manager Pod implements the functionality of PorterLB for the entire Kubernetes cluster.
- After the installation is complete, you can scale the porter-manager Deployment and assign multiple PorterLB replicas (porter-manager Pods) to multiple cluster nodes to ensure high availability. For details, see [Configure Multiple PorterLB Replicas](/docs/getting-started/configuration/configure-multiple-porter-replicas).

{{</ notice >}}

## Prerequisites

* You need to [prepare a K3s Kubernetes cluster](https://rancher.com/docs/k3s/latest/en/installation/), and ensure that the Kubernetes version is 1.15 or later. PorterLB requires CustomResourceDefinition (CRD) v1, which is only supported by Kubernetes 1.15 or later. PorterLB is designed to be used in bare-metal Kubernetes environments. However, you can also use a cloud-based K3s Kubernetes cluster for learning and testing.

* If you use Helm to install porter, ensure that the Helm version is Helm 3.

## Install PorterLB Using kubectl

1. Log in to the master node of the K3s Kubernetes cluster over SSH and run the following command:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubesphere/porter/master/deploy/porter.yaml
   ```
   
2. Run the following command to edit the porter-manager Deployment:

   ```bash
   kubectl edit deployment porter-manager -n porter-system
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

   By default, both PorterLB and K3s use port 443. Therefore, you need to change port 443 of PorterLB to avoid the port conflict. For details, see the [official K3s document](https://rancher.com/docs/k3s/latest/en/networking/#traefik-ingress-controller).

   {{</ notice >}}

4. Run the following command to check whether the status of porter-manager is **READY**: **1/1** and **STATUS**: **Running**. If yes, PorterLB has been installed successfully.

   ```bash
   kubectl get po -n porter-system
   ```

   ![verify-porter-kubectl](/images/en/docs/getting-started/installation/install-porter-on-k3s/verify-porter-kubectl.jpg)

## Delete PorterLB Using kubectl

1. To delete PorterLB, log in to the master node of the K3s Kubernetes cluster and run the following command:

   ```bash
   kubectl delete -f https://raw.githubusercontent.com/kubesphere/porter/master/deploy/porter.yaml
   ```

   {{< notice note >}}

   Before deleting PorterLB, you must first delete all Services that use PorterLB.

   {{</ notice >}}

2. Run the following command to check the result. If the porter-system namespace does not exist, PorterLB has been deleted successfully.

   ```bash
   kubectl get ns
   ```
   
   ![verify-porter-deletion-kubectl](/images/en/docs/getting-started/installation/install-porter-on-k3s/verify-porter-deletion-kubectl.jpg)

## Install PorterLB Using Helm

1. Log in to the master node of the K3s Kubernetes cluster over SSH and run the following command to configure the environment variable:

   ```bash
   export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
   ```

   {{< notice note >}}

   You need to configure this environment variable so that Helm can reach the K3s Kubernetes cluster. For details, see the [official K3s document](https://rancher.com/docs/k3s/latest/en/cluster-access/).

   {{</ notice >}}

2. Run the following commands to install PorterLB:

   ```bash 
   helm repo add test https://charts.kubesphere.io/test
   helm repo update
   helm install porter test/porter
   ```

3. Run the following command to edit the porter-manager Deployment:

   ```bash
   kubectl edit deployment porter-manager
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

   By default, both PorterLB and K3s use port 443. Therefore, you need to change port 443 of PorterLB to avoid the port conflict. For details, see the [official K3s document](https://rancher.com/docs/k3s/latest/en/networking/#traefik-ingress-controller).

   {{</ notice >}}

5. Run the following command to check whether the status of porter-manager is **READY**: **1/1** and **STATUS**: **Running**. If yes, PorterLB has been installed successfully.

   ```bash
   kubectl get po -A
   ```

   ![verify-porter-helm](/images/en/docs/getting-started/installation/install-porter-on-k3s/verify-porter-helm.jpg)

## Delete PorterLB Using Helm

1. To delete PorterLB, run the following command:

   ```bash
   helm delete porter
   ```

   {{< notice note >}}

   Before deleting PorterLB, you must first delete all Services that use PorterLB.

   {{</ notice >}}

2. Run the following command to check the result. If the PorterLB application does not exist, PorterLB has been deleted successfully.

   ```bash
   helm ls
   ```

   ![verify-porter-deletion-helm](/images/en/docs/getting-started/installation/install-porter-on-k3s/verify-porter-deletion-helm.jpg)