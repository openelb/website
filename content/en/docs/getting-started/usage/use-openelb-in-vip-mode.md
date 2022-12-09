---
title: "Use OpenELB in VIP Mode"
linkTitle: "Use OpenELB in VIP Mode"
weight: 3
---

This document demonstrates how to use OpenELB in VIP mode to expose a Service backed by two Pods. The Eip, Deployment and Service described in this document are examples only and you need to customize the commands and YAML configurations based on your requirements.

{{< notice note >}}

Ensure that the OpenELB version  is 0.5.0 or later.

{{</ notice >}}

## Prerequisites

* You need to [prepare a Kubernetes cluster where OpenELB has been installed](/docs/getting-started/installation/). All Kubernetes cluster nodes must be on the same Layer 2 network (under the same router).

* All Kubernetes cluster nodes must have only one NIC. The VIP mode currently does not support Kubernetes cluster nodes with multiple NICs.

* You need to prepare a client machine, which is used to verify whether OpenELB functions properly in Layer 2 mode. The client machine needs to be on the same network as the Kubernetes cluster nodes.

This document uses the following devices as an example:

| Device Name | IP Address  | MAC Address       | Description                 |
| ----------- | ----------- | ----------------- | --------------------------- |
| master1     | 192.168.0.2 | 52:54:22:a3:9a:d9 | Kubernetes cluster master   |
| worker-p001 | 192.168.0.3 | 52:54:22:3a:e6:6e | Kubernetes cluster worker 1 |
| worker-p002 | 192.168.0.4 | 52:54:22:37:6c:7b | Kubernetes cluster worker 2 |
| i-f3fozos0  | 192.168.0.5 | 52:54:22:fa:b9:3b | Client machine              |

## Step 1: Create an Eip Object

The Eip object functions as an IP address pool for OpenELB.

1. Run the following command to create a YAML file for the Eip object:

   ```bash
   vi vip-eip.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   apiVersion: network.kubesphere.io/v1alpha2
   kind: Eip
   metadata:
     name: vip-eip
   spec:
     address: 192.168.0.91-192.168.0.100
     protocol: vip
   ```

   {{< notice note >}}

   * The IP addresses specified in `spec:address` must be on the same network segment as the Kubernetes cluster nodes.

   * For details about the fields in the Eip YAML configuration, see [Configure IP Address Pools Using Eip](/docs/getting-started/configuration/configure-ip-address-pools-using-eip/).

   {{</ notice >}}

3. Run the following command to create the Eip object:

   ```bash
   kubectl apply -f vip-eip.yaml
   ```

## Step 2: Create a Deployment

The following creates a Deployment of two Pods using the luksa/kubia image. Each Pod returns its own Pod name to external requests. 

1. Run the following command to create a YAML file for the Deployment:

   ```bash
   vi vip-openelb.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: vip-openelb
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: vip-openelb
     template:
       metadata:
         labels:
           app: vip-openelb
       spec:
         containers:
           - image: luksa/kubia
             name: kubia
             ports:
               - containerPort: 8080
   ```

3. Run the following command to create the Deployment:

   ```bash
   kubectl apply -f vip-openelb.yaml
   ```

## Step 3: Create a Service

1. Run the following command to create a YAML file for the Service:

   ```bash
   vi vip-svc.yaml
   ```

2. Add the following information to the YAML file:

   ```yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: vip-svc
     annotations:
       lb.kubesphere.io/v1alpha1: openelb
       protocol.openelb.kubesphere.io/v1alpha1: vip
       eip.openelb.kubesphere.io/v1alpha2: vip-eip
   spec:
     selector:
       app: vip-openelb
     type: LoadBalancer
     ports:
       - name: http
         port: 80
         targetPort: 8080
     externalTrafficPolicy: Cluster
   ```

   {{< notice note >}}

   * You must set `spec:type` to `LoadBalancer`.
   * The `lb.kubesphere.io/v1alpha1: openelb` annotation specifies that the Service uses OpenELB.
   * The `protocol.openelb.kubesphere.io/v1alpha1: vip` annotation specifies that OpenELB is used in VIP mode.
   * The `eip.openelb.kubesphere.io/v1alpha2: vip-eip` annotation specifies the Eip object used by OpenELB. If this annotation is not configured, OpenELB automatically uses the first available Eip object that matches the protocol. You can also delete this annotation and add the `spec:loadBalancerIP` field (for example, `spec:loadBalancerIP: 192.168.0.91`) to assign a specific IP address to the Service.
   * If `spec:externalTrafficPolicy` is set to `Cluster` (default value), OpenELB randomly selects a node from all Kubernetes cluster nodes to handle Service requests. Pods on other nodes can also be reached over kube-proxy.
   * If `spec:externalTrafficPolicy` is set to `Local`, OpenELB randomly selects a node that contains a Pod in the Kubernetes cluster to handle Service requests. Only Pods on the selected node can be reached.

   {{</ notice >}}

3. Run the following command to create the Service:

      ```bash
      kubectl apply -f vip-svc.yaml
      ```

## Step 4: Verify OpenELB in VIP Mode

The following verifies whether OpenELB functions properly.

1. In the Kubernetes cluster, run the following command to obtain the external IP address of the Service:

   ```bash
   root@master1:~# kubectl get svc
   NAME         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
   kubernetes   ClusterIP      10.233.0.1      <none>         443/TCP        20h
   vip-svc   LoadBalancer   10.233.13.139   192.168.0.91   80:32658/TCP   14s
   ```

2. In the Kubernetes cluster, run the following command to obtain the IP addresses of the cluster nodes:

   ```bash
   root@master1:~# kubectl get nodes -o wide
   NAME    		STATUS   ROLES		AGE		VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
   master1   		Ready    master		20h		v1.17.9   192.168.0.2     <none>        Ubuntu 18.04.3 LTS   4.15.0-55-generic    docker://19.3.11
   worker-p001   	Ready    worker		20h		v1.17.9   192.168.0.3     <none>        Ubuntu 18.04.3 LTS   4.15.0-55-generic    docker://19.3.11
   worker-p002   	Ready    worker		20h		v1.17.9   192.168.0.4     <none>        Ubuntu 18.04.3 LTS   4.15.0-55-generic    docker://19.3.11
   ```

3. In the Kubernetes cluster, run the following command to check the nodes of the Pods:

   ```bash
   root@master1:~# kubectl get pod -o wide
   NAME                              READY   STATUS    RESTARTS   AGE     IP             NODE    		NOMINATED NODE   READINESS GATES
   vip-openelb-7b4fdf6f85-mnw5k   1/1     Running   0          3m27s   10.233.92.38   worker-p001   <none>           <none>
   vip-openelb-7b4fdf6f85-px4sm   1/1     Running   0          3m26s   10.233.90.31   worker-p002   <none>           <none>
   ```

   {{< notice note >}}

   In this example, the Pods are automatically assigned to different nodes. You can manually [assign Pods to different nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

   {{</ notice >}}

4. On the client machine, run the following commands to ping the Service IP address and check the IP neighbors:

   ```bash
   root@i-f3fozos0:~# ping 192.168.0.91 -c 4
   PING 192.168.0.91 (192.168.0.91) 56(84) bytes of data.
   64 bytes from 192.168.0.91: icmp_seq=1 ttl=64 time=0.162 ms
   64 bytes from 192.168.0.91: icmp_seq=2 ttl=64 time=0.119 ms
   64 bytes from 192.168.0.91: icmp_seq=3 ttl=64 time=0.145 ms
   64 bytes from 192.168.0.91: icmp_seq=4 ttl=64 time=0.123 ms
   
   --- 192.168.0.91 ping statistics ---
   4 packets transmitted, 4 received, 0% packet loss, time 3076ms
   rtt min/avg/max/mdev = 0.119/0.137/0.162/0.019 ms
   ```

   ```bash
   root@i-f3fozos0:~# ip neigh
   192.168.0.1 dev eth0 lladdr 02:54:22:99:ae:5d STALE
   192.168.0.2 dev eth0 lladdr 52:54:22:a3:9a:d9 STALE
   192.168.0.3 dev eth0 lladdr 52:54:22:3a:e6:6e STALE
   192.168.0.4 dev eth0 lladdr 52:54:22:37:6c:7b STALE
   192.168.0.91 dev eth0 lladdr 52:54:22:3a:e6:6e REACHABLE
   ```

   In the output of the `ip neigh` command, the MAC address of the Service IP address 192.168.0.91 is the same as that of worker-p001 192.168.0.3. Therefore, OpenELB has mapped the Service IP address to the MAC address of worker-p001.

5. On the client machine, run the following command to access the Service:

   ```bash
   curl 192.168.0.91
   ```

   If `spec:externalTrafficPolicy` in the [Service YAML configuration](#step-5-create-a-service) is set to `Cluster`, both Pods can be reached.

   ```bash
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-px4sm
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-mnw5k
   
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-px4sm
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-mnw5k
   ```
   
   If `spec:externalTrafficPolicy` in the [Service YAML configuration](#step-5-create-a-service) is set to `Local`, only the Pod on the node selected by OpenELB can be reached.
   
   ```bash
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-mnw5k
   
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-mnw5k
   
   root@i-f3fozos0:~# curl 192.168.0.91
   You've hit vip-openelb-7b4fdf6f85-mnw5k
   ```
   
   