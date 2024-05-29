---
title: "OpenELB IP address assignment"
linkTitle: "OpenELB IP address assignment"
weight: 1
---

This document demonstrates how OpenELB performs IP address management (IPAM) and shows how to properly configure services.

## 1. Specified Allocation

- Specified Eip Allocation:
    - Add annotations to the Service to specify that OpenELB is used as the load balancer plugin and ensure the specified Eip exists without needing to check if it is allocated to the namespace.

  ```yaml
  apiVersion: network.kubesphere.io/v1alpha2
  kind: Eip
  metadata:
    name: layer2-eip
  spec:
    address: 172.31.73.130-172.31.73.132
    namespaces: 
    - project
    interface: eth0
    protocol: layer2

  ---
  kind: Service
  apiVersion: v1
  metadata:
    name: nginx
    namespace: project-test
    annotations:
      lb.kubesphere.io/v1alpha1: openelb
      eip.openelb.kubesphere.io/v1alpha2: layer2-eip
  spec:
    selector:
      app: nginx
    type: LoadBalancer
    ports:
      - name: http
        port: 80
        targetPort: 80
    externalTrafficPolicy: Cluster
  ```


- Specified IP Allocation:
    - Add annotations to the Service to specify that OpenELB is used as the load balancer plugin and ensure the specified IP is within the CIDR of an existing Eip , without needing to check if the Eip is allocated to the namespace.
    - If the specified IP is already allocated, it will be shared.

  ```yaml
  apiVersion: network.kubesphere.io/v1alpha2
  kind: Eip
  metadata:
    name: layer2-eip
  spec:
    address: 172.31.73.130-172.31.73.132
    namespaces: 
    - project
    interface: eth0
    protocol: layer2

  ---
  kind: Service
  apiVersion: v1
  metadata:
    name: nginx
    namespace: project-test
    annotations:
      lb.kubesphere.io/v1alpha1: openelb
      eip.openelb.kubesphere.io/v1alpha1: "192.168.1.10"
  spec:
    selector:
      app: nginx
    type: LoadBalancer
    ports:
      - name: http
        port: 80
        targetPort: 80
    externalTrafficPolicy: Cluster


  # You can also set spec.loadBalancerIP to the specified ip. This method is recommended.
  ---
  kind: Service
  apiVersion: v1
  metadata:
    name: nginx1
    namespace: project-test
    annotations:
      lb.kubesphere.io/v1alpha1: openelb
  spec:
    loadBalancerIP: 192.168.1.10
    selector:
      app: nginx
    type: LoadBalancer
    ports:
      - name: http
        port: 80
        targetPort: 80
    externalTrafficPolicy: Cluster
  ```


## 2. Automatic Allocation

When you only configure `openelb` in the annotations without specifying the `eip` and `ip`, it enters automatic allocation. OpenELB will find a suitable eip first, then choose a suitable ip to complete the allocation.

Add annotations to the Service to specify that OpenELB is used as the load balancer plugin and ensure the namespace is allocated to available Eips.

Eip selection strategy during automatic allocation:
  - Preselect available Eips based on `eip.namespace` and `eip.namespaceSelector`.
  - Sort the preselected Eips by priority and choose the highest priority Eip .
  - If no Eips are preselected, use the default Eip .
  - Allocation fails and waits for a suitable Eip if no Eips are available.
  - Allocation fails and waits for IP release if all available Eips are fully allocated.

Example:
  ```yaml
  apiVersion: network.kubesphere.io/v1alpha2
  kind: Eip
  metadata:
    name: layer2-eip
  spec:
    address: 172.31.73.130-172.31.73.132
    namespaces: 
    - project
    interface: eth0
    protocol: layer2

  ---
  kind: Service
  apiVersion: v1
  metadata:
    name: nginx
    namespace: project
    annotations:
      lb.kubesphere.io/v1alpha1: openelb
  spec:
    selector:
      app: nginx
    type: LoadBalancer
    ports:
      - name: http
        port: 80
        targetPort: 80
    externalTrafficPolicy: Cluster
  ```

### There is a default eip in the cluster

Here is an example as shown in the figure: 

* The `default-eip` is set as the default eip using the annotation `"eip.openelb.kubesphere.io/is-default-eip": "true"`. 
* Namespace `namespace-1` and `namespace-2` have the label `label: test`, which is used by the `eip-selector`'s `namespaceSelector` to match and select. 
* The `eip-ns` Eip has a namespaces field that specifies `namespace-1`. This allows `eip-ns` to match and be allocated to `namespace-1`. 
* The `eip` Eip does not have any special configuration. It can only be used for specified allocation and cannot be automatically allocated.

![ipam-bind-to-namespace-1](/images/en/docs/getting-started/usage/ip-address-assignment/ipam-bind-to-namespace-1.svg)


| Namespace | Available Eip Count |
|--|--|
| `namespace-1` | 3 |
| `namespace-2` | 2 |
| `namespace-3` | 1 |

  {{< notice note >}}
  When the number of available Eips is greater than 1, the final Eip will be chosen according to the priorities. 
  * Namespace-bound Eips have higher priority than default Eips. 
  * For multiple namespace-bound Eips, priority will be determined by the priority field. Lower priority values indicate higher priority.
  {{</ notice >}}


### No default eip exists in the cluster

When there is no default Eip , the number of available Eips in `namespace-1`, `namespace-2`, and `namespace-3` will all decrease by 1. 

| Namespace | Available Eip Count |
|--|--|
| `namespace-1` | 2 |
| `namespace-2` | 1 |
| `namespace-3` | 0 |

In this case, `service` of type `loadbalancer` in `namespace-3` can only use manual specified allocation of Eips. Automatic Eip allocation is not possible for `service` since there are no available Eips in `namespace-3`.

![ipam-bind-to-namespace-2](/images/en/docs/getting-started/usage/ip-address-assignment/ipam-bind-to-namespace-2.svg)
