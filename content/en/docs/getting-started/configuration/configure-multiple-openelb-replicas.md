---
title: "Configure Multiple OpenELB Replicas"
linkTitle: "Configure Multiple OpenELB Replicas"
weight: 4
---

This document describes how to configure multiple openelb-speaker instances to ensure high availability in a production environment.

The `openelb-speaker` is deployed as a `DaemonSet`, which means an instance of `openelb-speaker` will be started on each node in the Kubernetes cluster.  If the number of nodes is large, the number of openelb-speaker instances will also be large.

In BGP mode, all openelb-speaker instances will respond to the BgpPeer configuration and attempt to establish a BGP connection with the peer BGP router by default. If the router is configured with BGP peer information for all nodes and establishes BGP connections with all of them, it can lead to significant load on the router.To mitigate this, you can use methods such as `nodeSelector`, `Node Affinity`, or `Taints and Tolerations` to schedule `openelb-speaker` only on specific nodes. This reduces the number of BGP connections and alleviates the load on the router. It is important to ensure that at least two instances of `openelb-speaker` are running under each router to maintain high availability.

In Layer2 or VIP mode, if you want only certain nodes to handle traffic, you can also use `nodeSelector`, `Node Affinity`, or `Taints and Tolerations` to schedule `openelb-speaker` on specific nodes. 

* If all Kubernetes cluster nodes are deployed under the same router, it is recommended to configure at least two openelb-speaker instances. These instances should be installed on two different Kubernetes cluster nodes to ensure redundancy and high availability.

* If the Kubernetes cluster nodes are deployed under different leaf routers, it is recommended to configure at least two openelb-speaker instances under each leaf router. This means one instance per node under each leaf router to ensure that each router has redundancy.


  {{< notice note >}}

  * If the Kubernetes cluster nodes are deployed under different routers, you need to perform further configuration so that the openelb-speaker instances establish BGP connections with the correct BGP routers. For details, see Configure OpenELB for Multi-router Clusters. In such a network topology, only BGP mode can be used. Layer2 mode is not suitable for clusters with nodes spread across different leaf routers.

  {{</ notice >}}


## Example Configuration

### Using Node Selector
First, label the target nodes:

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

Example:

```bash
kubectl label nodes node1 openelb-speaker=true
```

Then, update the DaemonSet configuration:


```yaml
spec:
  template:
    spec:
      nodeSelector:
        openelb-speaker: "true"
      ... ...
```

### Using Node Affinity
First, label the target nodes:


```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

Example:

```bash
kubectl label nodes node1 openelb-speaker=true
```
Then, update the DaemonSet configuration:


```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: openelb-speaker
                operator: In
                values:
                - "true"
      ... ...
```

### Using Taints and Tolerations
First, taint the target nodes:

```bash
kubectl taint nodes <node-name> key=value:NoSchedule
```

Example:

```bash
kubectl taint nodes node1 openelb-speaker=true:NoSchedule
```

Then, update the DaemonSet configuration:

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: "openelb-speaker"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
      ... ...
```

