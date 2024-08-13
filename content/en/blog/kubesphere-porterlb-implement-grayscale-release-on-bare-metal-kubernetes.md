---
title: "KubeSphere+PorterLB: Implement Grayscale Release on Bare-Metal Kubernetes"
linkTitle: "KubeSphere+PorterLB: Implement Grayscale Release on Bare-Metal Kubernetes"
description: Use KubeSphere and PorterLB to implement grayscale release on bare-metal Kubernetes.
keywords: PorterLB, Bare Metal, Kubernetes, KubeSphere, Gray Scale
author: Patrick
date: 2021-03-12
weight: 99999
---

[PorterLB](https://openelb.io) is a load balancer implementation designed for bare-metal Kubernetes clusters. As a sub-project of [KubeSphere](https://kubesphere.io), PorterLB fits well into the KubeSphere ecosystem. You can seamlessly integrate PorterLB as a plugin with KubeSphere to utilize the abundant features of the KubeSphere ecosystem.

During new feature release, the KubeSphere grayscale release feature allows users to freely distribute traffic among a stable version and a beta version of an application to both ensure service continuity and test the beta version before formally rolling it out.

In this article, I am going to introduce how to use KubeSphere and PorterLB to implement grayscale release for an application in a bare-metal Kubernetes cluster. To make you quickly understand how it works, I am going to directly use demonstration settings without digging too much into the details. You can obtain detailed guidance from the [KubeSphere documentation](https://kubesphere.io/docs/) and [PorterLB documentation](https://openelb.io/docs/).

## Architecture

![architecture](/images/en/blog/kubesphere-porterlb-implement-grayscale-release-on-bare-metal-kubernetes/architecture.png)

* Grayscale release

  In the preceding figure, an application Service backed by Pod v1 (stable version) is deployed in a Kubernetes cluster. After grayscale release is configured on KubeSphere, Pod v2 (beta version) is created and users can determine how much traffic is forwarded to Pod v1 and how much to Pod v2. After Pod v2 is fully tested, Pod v1 can be taken offline and Pod v2 will take over all traffic.

* KubeSphere gateway

  The application Service is exposed to the outside by using the KubeSphere gateway, which is in effect a Service backed by a Pod that functions as a reverse proxy. An external client uses a path of a domain name to access the application Service. The reverse proxy Pod obtains the mapping between the path and the application Service from a Route object.

* PorterLB

  PorterLB installed in the Kubernetes cluster sends an ARP packet to the router, and tells the router to forward traffic destined for the gateway Service to node 1. If node 1 fails, the traffic will be forwarded to node 2.

## Procedure

### Prerequisites

* You need to prepare a Kubernetes cluster, and install [KubeSphere](https://kubesphere.io/docs/installing-on-kubernetes/) and [PorterLB](https://openelb.io/docs/getting-started/installation/) in the Kubernetes cluster.
* On KubeSphere, you need to [create a project and an account](https://kubesphere.io/docs/quick-start/create-workspace-and-project/). The role of the account in the project must be `project-admin`.

### Operations

Step 1: Set the KubeSphere gateway to use PorterLB and create an Eip object.
1. Log in to KubeSphere and go to your project.

2. Choose **Project Settings** > **Advanced Settings** on the left navigation bar and click **Set Gateway** on the right.

3. Click **LoadBalancer**, set **Application Governance** to **On**, add the following annotations, and click **Save**. The annotations set the KubeSphere gateway Service to use PorterLB in Layer 2 mode.

   ```yaml
   lb.kubesphere.io/v1alpha1: porter
   protocol.porter.kubesphere.io/v1alpha1: layer2
   eip.porter.kubesphere.io/v1alpha2: porter-layer2-eip # Name of the Eip object.
   ```

4. Move the cursor to <img src="/images/en/blog/kubesphere-porterlb-implement-grayscale-release-on-bare-metal-kubernetes/kubectl.png" width="25px"> in the lower-right corner and click **Kubectl** to open the CLI.

5. Run the `vi porter-layer2-eip.yaml` command to create a YAML file for an Eip object and add the following information to the YAML file:
   ```yaml
   apiVersion: network.kubesphere.io/v1alpha2
   kind: Eip
   metadata:
     name: porter-layer2-eip
   spec:
     # Use an unoccupied address on the same network segment as your K8s cluster.
     address: 192.168.0.100
     interface: eth0
     protocol: layer2
   ```
   
6. Run the `kubectl apply -f eip.yaml` command to create the Eip object, which functions as an IP address pool for PorterLB.

Step 2: Create an application.
1. Choose **Application Workloads** on the left navigation bar and click **Create Composing Application** on the right.
2. Set **Application Name** to **demo-app**, **Application Version(Optional)** to **v1**, **Application Governance** to **On**, and click **Next**.
3. Click **Add Service**, click **Stateless Service**, set **Name** to `demo-svc`, and click **Next**.
5. Click **Add Container Image**, set **Image** to `luksa/kubia`, **Container Port** to `8080`, **Service Port** to `80`, click **√**, and click **Next**.
6. Click **Next** on the **Mount Volumes** tab, click **Add** on the **Advanced Settings** tab, and click **Next**.
7. Click **Add Route Rule**, click **Specify Domain**, set **HostName** to `demo.com`, **Paths** to `/path | demo-svc | 80`, and click **OK**.
8. Click **Create**.

Step 3: Configure grayscale release.
1. Choose **Grayscale Release** on the left navigation bar, move the cursor to **Canary Release**, and click **Create Job**.
2. Set **Release Job Name** to `demo-canary` and click **Next**.
3. Select `demo-app` from the drop-down list, click **Select** on the right of **demo-svc**, and click **Next**.
4. Set **Grayscale Release Version Number** to `v2` and click **Next**.
5. Click **Create** on the **Policy Config** tab.

Step 4: Test grayscale release.

1. Choose **Grayscale Release** on the left navigation bar, click the **Job Status** tab, and click **demo-canary** on the right. 

2. In the **Real-time traffic distribution** area, move the slider so that 100% traffic is sent to v2.

3. Log in to a client machine connected to the gateway Service IP address (configured in the Eip object) and add the domain name information to the `etc/hosts` file:

   ``` bash
   192.168.0.100 demo.com
   ```

4. On the client machine, run the `curl demo.com/path` command for multiple times to access the application Service.

   If grayscale release functions properly, only v2 can be accessed.

   <img src="/images/en/blog/kubesphere-porterlb-implement-grayscale-release-on-bare-metal-kubernetes/v2.png" width="600px">

5. On KubeSphere, repeat step 2 so that 100% traffic is sent to v1.

6. On the client machine, run the `curl demo.com/path` command for multiple times to access the application Service.

   If grayscale release functions properly, only v1 can be accessed.

   <img src="/images/en/blog/kubesphere-porterlb-implement-grayscale-release-on-bare-metal-kubernetes/v1.png" width="600px">

7. After v2 is fully tested, you can set v2 to take over all traffic and take the canary release job offline to formally release v2.

   <img src="/images/en/blog/kubesphere-porterlb-implement-grayscale-release-on-bare-metal-kubernetes/job-offline.png">

