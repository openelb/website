---
title: "Install OpenELB on KubeSphere"
linkTitle: "Install OpenELB on KubeSphere"
weight: 2
---

This document describes how to install and delete OpenELB on the [KubeSphere](https://kubesphere.io/) web console.

{{< notice note >}}

- In a Kubernetes cluster, you only need to install OpenELB once. After the installation is complete, an openelb-controller Deployment containing an openelb-controller Pod and an openelb-speaker DaemonSet containing openelb-speaker Pods will be installed in the cluster. The openelb-controller Pod is used to implement the IPAM for service load balancer IPs, while the openelb-speaker Pods are used to announce the service load balancer IPs.

- After the installation is complete, you can set node selectors to ensure that the load traffic runs on specific nodes. For details, see [Configure Multiple OpenELB Replicas](/docs/getting-started/configuration/configure-multiple-openelb-replicas).

{{</ notice >}}

## Prerequisites

You need to prepare a Kubernetes cluster with KubeSphere, and ensure that the Kubernetes version is 1.15 or later. OpenELB requires CustomResourceDefinition (CRD) v1, which is only supported by Kubernetes 1.15 or later. You can use the following methods to install KubeSphere:

* [Deploy a new Kubernetes cluster with KubeSphere](https://kubesphere.io/docs/installing-on-linux/).
* [Install KubeSphere in an existing Kubernetes cluster](https://kubesphere.io/docs/installing-on-kubernetes/).

OpenELB is designed to be used in bare-metal Kubernetes environments. However, you can also use a cloud-based Kubernetes cluster for learning and testing.

## Install OpenELB on the KubeSphere Web Console

1. Log in to the KubeSphere console.

2. On the top navigation bar, choose **App Store**, and search **openelb**.

   ![search-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/search-openelb.jpg)

7. Click **Install** and follow the wizard instructions to complete the installation. You can customize the chart configuration in the YAML file based on your requirements.

   ![deploy-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/deploy-openelb.png)

   ![openelb-setting](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/openelb-setting.png)

   ![openelb-yaml](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/openelb-yaml.png)

8. Choose **Application Workloads** > **Pods** on the left navigation bar to check whether the status of openelb-controller and openelb-speaker is **running**. If yes, OpenELB has been installed successfully.

   ![verify-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/verify-openelb.png)

## Delete OpenELB on the KubeSphere Web Console

To delete OpenELB on the KubeSphere web console, go to your project, choose **Application Workloads** > **Apps** on the left navigation bar, click ![openelb-operation](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/openelb-operation.jpg) on the right of the OpenELB application, and choose **Delete** from the drop-down list.

![delete-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/delete-openelb.png)

{{< notice note >}}

Before deleting OpenELB, you should first delete all Services that use OpenELB.

{{</ notice >}}