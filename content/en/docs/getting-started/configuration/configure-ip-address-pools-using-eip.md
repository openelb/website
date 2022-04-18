---
title: "Configure IP Address Pools Using Eip"
linkTitle: "Configure IP Address Pools Using Eip"
weight: 1
---

This document describes how to configure an Eip object, which functions as an IP address pool for OpenELB in BGP mode, Layer 2 mode, and VIP mode.

OpenELB assigns IP addresses in Eip objects to LoadBalancer Services in the Kubernetes cluster.

{{< notice note >}}

Currently, OpenELB supports only IPv4 and will soon support IPv6.

{{</ notice >}}

## Configure an Eip Object for OpenELB

You can create an Eip object to provide an IP address pool for OpenELB. The following is an example of the Eip YAML configuration:

```yaml
apiVersion: network.kubesphere.io/v1alpha2
kind: Eip
metadata:
    name: eip-sample-pool
    annotations:
      eip.openelb.kubesphere.io/is-default-eip: "true"
spec:
    address: 192.168.0.91-192.168.0.100
    protocol: layer2
    interface: eth0
    disable: false
status:
    occupied: false
    usage: 1
    poolSize: 10
    used: 
      "192.168.0.91": "default/test-svc"
    firstIP: 192.168.0.91
    lastIP: 192.168.0.100
    ready: true
    v4: true
```

The fields are described as follows:

`metadata`:

* `name`: Name of the Eip object.

* `annotations`:

  * `eip.openelb.kubesphere.io/is-default-eip`: Whether the current Eip object is the default Eip object. The value can be `"true"` or `"false"`. If a default Eip object exists, you do not need to specify an Eip object when creating a Service and the system automatically assigns an IP address from the default Eip object to the Service. If you do not specify an Eip object when creating a Service while no default IP address exists, the system automatically assigns an IP address from any available Eip object, if any, to the Service. For each Kubernetes cluster, you can set only one Eip object as the default Eip object.

`spec`:

* `address`: One or more IP addresses, which will be used by OpenELB. The value format can be:
  
  * `IP address`, for example, `192.168.0.100`.
  * `IP address/Subnet mask`, for example, `192.168.0.0/24`.
  * `IP address 1-IP address 2`, for example, `192.168.0.91-192.168.0.100`.
  
  
  {{< notice note >}}
  
  IP segments in different Eip objects cannot overlap. Otherwise, a resource creation error will occur.
  
  {{</ notice >}}


* `protocol`: Specifies which mode of OpenELB the Eip object is used for. The value can be `bgp`, `layer2`, or `vip`. If this field is not specified, the default value `bgp` is used.

* `interface`: NIC on which OpenELB listens for ARP or NDP requests. This field is valid only when `protocol` is set to `layer2`.

  {{< notice tip >}}

  If the NIC names of the Kubernetes cluster nodes are different, you can set the value to `can_reach:IP address` (for example, `can_reach:192.168.0.5`) so that OpenELB automatically obtains the name of the NIC that can reach the IP address. In this case, you must ensure that the IP address is not used by Kubernetes cluster nodes but can be reached by the cluster nodes.

  {{</ notice >}}

* `disable`: Specifies whether the Eip object is disabled. The value can be:
  
  * `false`: OpenELB can assign IP addresses in the Eip object to new LoadBalancer Services.
  * `true`: OpenELB stops assigning IP addresses in the Eip object to new LoadBalancer Services. Existing Services are not affected.

`status`: Fields under `status` specify the status of the Eip object and are automatically configured. When creating an Eip object, you do not need to configure these fields.

* `occupied`: Specifies whether IP addresses in the Eip object have been used up.

* `usage`: Specifies how many IP addresses in the Eip object have been assigned to Services.
* `used`: Specifies the used IP addresses and the Services that use the IP addresses. The Services are displayed in the `Namespace/Service name` format (for example, `default/test-svc`).

* `poolSize`: Total number of IP addresses in the Eip object.

* `firstIP`: First IP address in the Eip object.

* `lastIP`: Last IP address in the Eip object.

* `v4`: Specifies whether the address family is IPv4. Currently, OpenELB supports only IPv4 and the value can only be `true`.

* `ready`: Specifies whether the Eip-associated program used for BGP/ARP/NDP routes publishing has been initialized. The program is integrated in OpenELB.
