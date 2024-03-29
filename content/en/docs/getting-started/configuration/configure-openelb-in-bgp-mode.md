---
title: "Configure OpenELB in BGP Mode"
linkTitle: "Configure OpenELB in BGP Mode"
weight: 2
---

This document describes how to configure OpenELB in BGP mode. If OpenELB is used in Layer 2 mode, you do not need to configure OpenELB.

## Configure Local BGP Properties Using BgpConf

You can create a BgpConf object in the Kubernetes cluster to configure the local BGP properties on OpenELB. The following is an example of the BgpConf YAML configuration:

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

The fields are described as follows:

`metadata`:

* `name`: BgpConf object name. OpenELB recognizes only the name `default`. BgpConf objects with other names will be ignored.

`spec`:

* `as`: Local ASN, which must be different from the value of `spec:conf:peerAS` in the BgpPeer configuration.
* `listenPort`: Port on which OpenELB listens. The default value is `179` (default BGP port number). If other components (such as Calico) in the Kubernetes cluster also use BGP and port 179, you must set a different value to avoid the conflict.
* `routerID`: Local router ID, which is usually set to the IP address of the master NIC of the Kubernetes master node. If this field is not specified, the first IP address of the node where openelb-manager is located will be used.

## Configure Peer BGP Properties Using BgpPeer

You can create a BgpPeer object in the Kubernetes cluster to configure the peer BGP properties on OpenELB. The following is an example of the BgpPeer YAML configuration:

```yaml
apiVersion: network.kubesphere.io/v1alpha2
kind: BgpPeer
metadata:
  name: bgppeer-sample
spec:
  conf:
    peerAs: 50001
    neighborAddress: 192.168.0.5
  afiSafis:
    - config:
        family:
          afi: AFI_IP
          safi: SAFI_UNICAST
        enabled: true
      addPaths:
        config:
          sendMax: 10
  nodeSelector:
    matchLabels:
      openelb.kubesphere.io/rack: leaf1
```

The fields are described as follows:

`metadata`:

* `name`: Name of the BgpPeer object. If there are multiple peer BGP routers, you can create multiple BgpPeer objects with different names.

`spec:conf`:

* `peerAS`: ASN of the peer BGP router, which must be different from the value of `spec:as` in the BgpConf configuration.
* `neighborAddress`: IP address of the peer BGP router.

`spec:afiSafis:addPaths:config`:

* `sendMax`: Maximum number of equivalent routes that OpenELB can send to the peer BGP router for Equal-Cost Multi-Path (ECMP) routing. The default value is `10`.

`spec:nodeSelector:matchLabels`:

* `openelb.kubesphere.io/rack`: If the Kubernetes cluster nodes are deployed under different routers and each node has one OpenELB replica, you need to configure this field so that the OpenELB replica on the correct node establishes a BGP connection with the peer BGP router. By default, all openelb-manager replicas will respond to the BgpPeer configuration and attempt to establish a BGP connection with the peer BGP router.

Other fields under `spec:afiSafis` specify the address family. Currently, OpenELB supports only IPv4 and you can directly use the values in the example configuration.