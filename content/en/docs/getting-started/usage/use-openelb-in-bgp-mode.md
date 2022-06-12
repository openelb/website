---
title: "Use OpenELB in BGP Mode"
linkTitle: "Use OpenELB in BGP Mode"
weight: 1
---

This document demonstrates how to use OpenELB in BGP mode to expose a Service backed by two Pods. The BgpConf, BgpPeer, Eip, Deployment and Service described in this document are examples only and you need to customize the commands and YAML configurations based on your requirements.

Instead of using a real router, this document uses a Linux server with [BIRD](https://bird.network.cz/) to simulate a router so that users without a real router can also use OpenELB in BGP mode for tests.

## Prerequisites

* You need to [prepare a Kubernetes cluster where OpenELB has been installed](/docs/getting-started/installation/).
* You need to prepare a Linux server that communicates with the Kubernetes cluster properly. BIRD will be installed on the server to simulate a BGP router. 
* If you use a real router instead of BIRD, the router must support BGP and Equal-Cost Multi-Path (ECMP) routing. In addition, the router must also support receiving multiple equivalent routes from the same neighbor.

This document uses the following devices as an example:

| Device Name | IP Address  | Description                                                  |
| ----------- | ----------- | ------------------------------------------------------------ |
| master1     | 192.168.0.2 | Kubernetes cluster master, where OpenELB is installed.       |
| worker-p001 | 192.168.0.3 | Kubernetes cluster worker 1                                  |
| worker-p002 | 192.168.0.4 | Kubernetes cluster worker 2                                  |
| i-f3fozos0  | 192.168.0.5 | BIRD machine, where BIRD will be installed to simulate a BGP router. |

## Step 1: Install and Configure BIRD

If you use a real router, you can skip this step and perform configuration on the router instead.

1. Log in to the BIRD machine and run the following commands to install BIRD:

   ```bash
   sudo add-apt-repository ppa:cz.nic-labs/bird
   sudo apt-get update 
   sudo apt-get install bird
   sudo systemctl enable bird 
   ```

   {{< notice note >}}

   * BIRD 1.5 does not support ECMP. To use all features of OpenELB, you are advised to install BIRD 1.6 or later.
   * The preceding commands apply only to Debian-based OSs such as Debian and Ubuntu. On Red Hat-based OSs such as RHEL and CentOS, use [yum](https://access.redhat.com/solutions/9934) instead.
   * You can also install BIRD according to the [official BIRD documentation](https://bird.network.cz/).

   {{</ notice >}}

2. Run the following command to edit the BIRD configuration file:

   ```bash
   vi /etc/bird/bird.conf
   ```

3. Configure the BIRD configuration file as follows:

   ```conf
   router id 192.168.0.5;
   
   protocol kernel {
       scan time 60;
       import none;
       export all;
       merge paths on;
   }
   
   protocol bgp neighbor1 {
       local as 50001;
       neighbor 192.168.0.2 port 17900 as 50000;
       source address 192.168.0.5;
       import all;
       export all;
       enable route refresh off;
       add paths on;
   }
   ```

   {{< notice note >}}

   * For test usage, you only need to customize the following fields in the preceding configuration:

     `router id`: Router ID of the BIRD machine, which is usually set to the IP address of the BIRD machine.

     `protocol bgp neighbor1`:

     * `local as`: ASN of the BIRD machine, which must be different from the ASN of the Kubernetes cluster.
     * `neighbor`: Master node IP address, BGP port number, and ASN of the Kubernetes cluster. Use port `17900` instead of the default BGP port `179` to avoid conflicts with other BGP components in the system.
     * `source address`: IP address of the BIRD machine.

   * If multiple nodes in the Kubernetes are used as BGP neighbors, you need to configure multiple BGP neighbors in the BIRD configuration file. 

   * For details about the BIRD configuration file, see the [official BIRD documentation](https://bird.network.cz/).

   {{</ notice >}}

4. Run the following command to restart BIRD:

   ```bash
   sudo systemctl restart bird 
   ```

5. Run the following command to check whether the status of BIRD is active:

   ```bash
   sudo systemctl status bird
   ```

   ![bird-status](/images/en/docs/getting-started/usage/use-openelb-in-bgp-mode/bird-status.jpg)

   {{< notice note >}}

   If the status of BIRD is not active, you can run the following command to check the error logs:

   ```bash
   journalctl -f -u bird
   ```

   {{</ notice >}}

## Step 2: Create a BgpConf Object

The BgpConf object is used to configure the local (Kubernetes cluster) BGP properties on OpenELB.

1. Run the following command to create a YAML file for the BgpConf object:

   ```bash
   vi bgp-conf.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   apiVersion: network.kubesphere.io/v1alpha2
   kind: BgpConf
   metadata:
     name: default
   spec:
     as: 50000
     listenPort: 17900
     routerId: 192.168.0.2
   ```

   {{< notice note >}}

   For details about the fields in the BgpConf YAML configuration, see [Configure Local BGP Properties Using BgpConf](/docs/getting-started/configuration/configure-openelb-in-bgp-mode/#configure-local-bgp-properties-using-bgpconf).

   {{</ notice >}}

3. Run the following command to create the BgpConf object:

   ```bash
   kubectl apply -f bgp-conf.yaml
   ```

## Step 3: Create a BgpPeer Object

The BgpPeer object is used to configure the peer (BIRD machine) BGP properties on OpenELB.

1. Run the following command to create a YAML file for the BgpPeer object:

   ```bash
   vi bgp-peer.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   apiVersion: network.kubesphere.io/v1alpha2
   kind: BgpPeer
   metadata:
     name: bgp-peer
   spec:
     conf:
       peerAs: 50001
       neighborAddress: 192.168.0.5
   ```

   {{< notice note >}}

   For details about the fields in the BgpPeer YAML configuration, see [Configure Peer BGP Properties Using BgpPeer](/docs/getting-started/configuration/configure-openelb-in-bgp-mode/#configure-peer-bgp-properties-using-bgppeer).

   {{</ notice >}}

3. Run the following command to create the BgpPeer object:

   ```bash
   kubectl apply -f bgp-peer.yaml
   ```

## Step 4: Create an Eip Object

The Eip object functions as an IP address pool for OpenELB.

1. Run the following command to create a YAML file for the Eip object:

   ```bash
   vi bgp-eip.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   apiVersion: network.kubesphere.io/v1alpha2
   kind: Eip
   metadata:
     name: bgp-eip
   spec:
     address: 172.22.0.2-172.22.0.10
   ```

   {{< notice note >}}

   The address can be any random but valid IP address representation (one or more), execpt the IP addresses already in use.
   For details about the fields in the Eip YAML configuration, see [Configure IP Address Pools Using Eip](/docs/getting-started/configuration/configure-ip-address-pools-using-eip/).

   {{</ notice >}}

3. Run the following command to create the Eip object:

   ```bash
   kubectl apply -f bgp-eip.yaml
   ```

## Step 5: Create a Deployment

The following creates a Deployment of two Pods using the luksa/kubia image. Each Pod returns its own Pod name to external requests.

1. Run the following command to create a YAML file for the Deployment:

   ```bash
   vi bgp-openelb.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: bgp-openelb
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: bgp-openelb
     template:
       metadata:
         labels:
           app: bgp-openelb
       spec:
         containers:
           - image: luksa/kubia
             name: kubia
             ports:
               - containerPort: 8080
   ```

3. Run the following command to create the Deployment:

   ```bash
   kubectl apply -f bgp-openelb.yaml
   ```

## Step 6: Create a Service

1. Run the following command to create a YAML file for the Service:

   ```bash
   vi bgp-svc.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: bgp-svc
     annotations:
       lb.kubesphere.io/v1alpha1: openelb
       protocol.openelb.kubesphere.io/v1alpha1: bgp
       eip.openelb.kubesphere.io/v1alpha2: bgp-eip
   spec:
     selector:
       app: bgp-openelb
     type: LoadBalancer
     ports:
       - name: http
         port: 80
         targetPort: 8080
     externalTrafficPolicy: Cluster
   ```

   {{< notice note >}}

   - You must set `spec:type` to `LoadBalancer`.
   - The `lb.kubesphere.io/v1alpha1: openelb` annotation specifies that the Service uses OpenELB.
   - The `protocol.openelb.kubesphere.io/v1alpha1: bgp` annotation specifies that OpenELB is used in BGP mode.
   - The `eip.openelb.kubesphere.io/v1alpha2: bgp-eip` annotation specifies the Eip object used by OpenELB. If this annotation is not configured, OpenELB automatically uses the first available Eip object that matches the protocol. You can also delete this annotation and add the `spec:loadBalancerIP` field (for example, `spec:loadBalancerIP: 172.22.0.2`) to assign a specific IP address to the Service.
   - In the BGP mode, you can set `spec:loadBalancerIP` of multiple Services to the same value for IP address sharing (the Services are distinguished by different Service ports). In this case, you must set `spec:ports:port` to different values and `spec:externalTrafficPolicy` to `Cluster` for the Services. 
   - If `spec:externalTrafficPolicy` is set to `Cluster` (default value), OpenELB uses all Kubernetes cluster nodes as the next hops destined for the Service.
   - If `spec:externalTrafficPolicy` is set to `Local`, OpenELB uses only Kubernetes cluster nodes that contain Pods as the next hops destined for the Service.

   {{</ notice >}}

3. Run the following command to create the Service:

   ```bash
   kubectl apply -f bgp-svc.yaml
   ```

## Step 7: Verify OpenELB in BGP Mode

The following verifies whether OpenELB functions properly.

1. In the Kubernetes cluster, run the following command to obtain the external IP address of the Service:

   ```bash
   root@master1:~# kubectl get svc
   NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
   kubernetes   ClusterIP      10.233.0.1     <none>        443/TCP        20h
   bgp-svc      LoadBalancer   10.233.15.12   172.22.0.2    80:32278/TCP   6m9s
   ```

2. In the Kubernetes cluster, run the following command to obtain the IP addresses of the cluster nodes:

   ```bash
   root@master1:~# kubectl get nodes -o wide
   NAME    		STATUS   ROLES		AGE		VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
   master1   		Ready    master		20h		v1.17.9   192.168.0.2     <none>        Ubuntu 18.04.3 LTS   4.15.0-55-generic    docker://19.3.11
   worker-p001   	Ready    worker		20h		v1.17.9   192.168.0.3     <none>        Ubuntu 18.04.3 LTS   4.15.0-55-generic    docker://19.3.11
   worker-p002   	Ready    worker		20h		v1.17.9   192.168.0.4     <none>        Ubuntu 18.04.3 LTS   4.15.0-55-generic    docker://19.3.11
   ```

3. On the BIRD machine, run the following command to check the routing table. If equivalent routes using the Kubernetes cluster nodes as next hops destined for the Service are displayed, OpenELB functions properly.

   ```bash
   ip route
   ```

   If `spec:externalTrafficPolicy` in the [Service YAML configuration](#step-6-create-a-service) is set to `Cluster`, all Kubernetes cluster nodes are used as the next hops.

   ```bash
   default via 192.168.0.1 dev eth0
   172.22.0.2 proto bird metric 64
           nexthop via 192.168.0.2 dev eth0 weight 1
           nexthop via 192.168.0.3 dev eth0 weight 1
           nexthop via 192.168.0.4 dev eth0 weight 1
   192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.5
   ```

   If `spec:externalTrafficPolicy` in the [Service YAML configuration](#step-6-create-a-service) is set to `Local`, only Kubernetes cluster nodes that contain Pods are used as the next hops.

   ```bash
   default via 192.168.0.1 dev eth0
   172.22.0.2 proto bird metric 64
           nexthop via 192.168.0.3 dev eth0 weight 1
           nexthop via 192.168.0.4 dev eth0 weight 1
   192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.5
   ```

   

4. On the BIRD machine, run the `curl` command to access the Service:

   ```bash
   root@i-f3fozos0:~# curl 172.22.0.2
   You've hit bgp-openelb-648bcf8d7c-86l8k
   
   root@i-f3fozos0:~# curl 172.22.0.2
   You've hit bgp-openelb-648bcf8d7c-pngxj
   ```


