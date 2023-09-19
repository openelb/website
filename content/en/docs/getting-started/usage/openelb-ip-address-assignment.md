---
title: "OpenELB IP address assignment"
linkTitle: "OpenELB IP address assignment"
weight: 1
---

This document demonstrates how OpenELB performs IP address management (IPAM) and shows how to properly configure services.

## 1. Specified Allocation

Specified allocation first finds the specified `eip`, then allocates an available IP from that `eip`. Specify the following parameters in the created loadbalancer service:

* Specify EIP allocation

```yaml
annotations:
  "lb.kubesphere.io/v1alpha1": "openelb"
  "eip.openelb.kubesphere.io/v1alpha2": "eip-name"
```

* Specify IP allocation

```yaml
annotations:
  "lb.kubesphere.io/v1alpha1": "openelb"
  "eip.openelb.kubesphere.io/v1alpha1": "192.168.1.10"
```

## 2. Automatic Allocation

When we only configure `openelb` in the annotations without specifying the `eip` and `ip`, it enters automatic allocation. OpenELB will similarly find a suitable eip first, then choose a suitable ip to complete the allocation:

```yaml
annotations:
  "lb.kubesphere.io/v1alpha1": "openelb"
```

### There is a default eip in the cluster

Here is an example to illustrate. As shown in the figure: 

* The `default-eip` is set as the default eip using the annotation `"eip.openelb.kubesphere.io/is-default-eip": "true"`. 
* Namespace `namespace-1` and `namespace-2` have the label `label: test`, which is used by the `eip-selector`'s `namespaceSelector` to match and select. 
* The `eip-ns` EIP has a namespaces field that specifies `namespace-1`. This allows `eip-ns` to match and be allocated to `namespace-1`. 
* The `eip` EIP does not have any special configuration. It can only be used for specified allocation and cannot be automatically allocated.

![ipam-bind-to-namespace-1](/images/en/docs/getting-started/usage/ip-address-assignment/ipam-bind-to-namespace-1.svg)


| Namespace | Available EIP Count |
|--|--|
| `namespace-1` | 3 |
| `namespace-2` | 2 |
| `namespace-3` | 1 |

  {{< notice note >}}
  When the number of available EIPs is greater than 1, there will be some priority selection to choose the final EIP to use. 
  * Namespace-bound EIPs have higher priority than default EIPs. 
  * For multiple namespace-bound EIPs, priority will be determined by the priority field. Lower priority values indicate higher priority.
  {{</ notice >}}


### No default eip exists in the cluster

When there is no default EIP, the number of available EIPs in `namespace-1`, `namespace-2`, and `namespace-3` will all decrease by 1. 

| Namespace | Available EIP Count |
|--|--|
| `namespace-1` | 2 |
| `namespace-2` | 1 |
| `namespace-3` | 0 |

In this case, `service` of type `loadbalancer` in `namespace-3` can only use manual specified allocation of EIPs. Automatic EIP allocation is not possible for `service` since there are no available EIPs in `namespace-3`.

![ipam-bind-to-namespace-2](/images/en/docs/getting-started/usage/ip-address-assignment/ipam-bind-to-namespace-2.svg)
