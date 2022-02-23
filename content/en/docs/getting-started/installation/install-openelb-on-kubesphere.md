---
title: "Install OpenELB on KubeSphere"
linkTitle: "Install OpenELB on KubeSphere"
weight: 2
---

This document describes how to install and delete OpenELB on the [KubeSphere](https://kubesphere.io/) web console.

{{< notice note >}}

- In a Kubernetes cluster, you only need to install OpenELB once. After the installation is complete, a openelb-manager Deployment that contains a openelb-manager Pod is installed in the cluster. The openelb-manager Pod implements the functionality of OpenELB for the entire Kubernetes cluster.
- After the installation is complete, you can scale the openelb-manager Deployment and assign multiple OpenELB replicas (openelb-manager Pods) to multiple cluster nodes to ensure high availability. For details, see [Configure Multiple OpenELB Replicas](/docs/getting-started/configuration/configure-multiple-openelb-replicas/).

{{</ notice >}}

## Prerequisites

You need to prepare a Kubernetes cluster with KubeSphere, and ensure that the Kubernetes version is 1.15 or later. OpenELB requires CustomResourceDefinition (CRD) v1, which is only supported by Kubernetes 1.15 or later. You can use the following methods to install KubeSphere:

* [Deploy a new Kubernetes cluster with KubeSphere](https://kubesphere.io/docs/installing-on-linux/).
* [Install KubeSphere in an existing Kubernetes cluster](https://kubesphere.io/docs/installing-on-kubernetes/).

OpenELB is designed to be used in bare-metal Kubernetes environments. However, you can also use a cloud-based Kubernetes cluster for learning and testing.

## Install OpenELB on the KubeSphere Web Console

1. Log in to the KubeSphere console and go to your workspace.

   ![enter-workspace](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/enter-workspace.jpg)

2. On the left navigation bar, choose **Apps Management** > **App Repos**, and click **Add Repo** on the right.

   ![add-repo](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/add-repo.jpg)

3. In the displayed dialog box, set **App Repository Name** (for example, `KubeSphere-test`), set **URL** to `https://charts.kubesphere.io/test`, click **Validate** to check the URL, and click **OK**.

   ![repo-spec](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/repo-spec.jpg)

4. Go to your project, choose **Application Workloads** > **Applications** on the left navigation bar, and click **Deploy New Application** on the right.

   ![deploy-new-app](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/deploy-new-app.jpg)

5. In the displayed dialog box, click **From App Templates**.

   ![from-app-templates](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/from-app-templates.jpg)

6. Select **KubeSphere-test** from the drop-down list and click **porter**.

   ![openelb-template](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/porter-template.jpg)

7. Click **Deploy** and follow the wizard instructions to complete the installation. You can customize the chart configuration in the YAML file based on your requirements.

   ![deploy-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/deploy-porter.jpg)

   ![porter-yaml](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/porter-yaml.jpg)

8. Choose **Application Workloads** > **Pods** on the left navigation bar to check whether the status of openelb-manager is **running**. If yes, OpenELB has been installed successfully.

   ![verify-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/verify-porter.jpg)

## Delete OpenELB on the KubeSphere Web Console

To delete OpenELB on the KubeSphere web console, go to your project, choose **Application Workloads** > **Applications** on the left navigation bar, click ![porter-operation](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/porter-operation.jpg) on the right of the OpenELB application, and choose **Delete** from the drop-down list.

![delete-openelb](/images/en/docs/getting-started/installation/install-openelb-on-kubesphere/delete-porter.jpg)

{{< notice note >}}

Before deleting OpenELB, you must first delete all Services that use OpenELB.

{{</ notice >}}