---
title: "Concepts"
linkTitle: "Concepts"
weight: 2
description: >
  The basic concepts of Porter project.
---

{{% pageinfo %}}
This guide elaborates on the basic concepts and principles of Porter.
{{% /pageinfo %}}

## Principle

The following figure describes the principle of Porter. Assuming there is a distributed service deployed on node1 (192.168.0.2) and node2 (192.168.0.6). The service needs to be accessed through EIP `1.1.1.1`. After deploying the [Example Service](https://github.com/kubesphere/porter/blob/master/test/samples/test.yaml), Porter will automatically synchronize routing information to the leaf switch, and then synchronize to the spine and border switch, thus external users can access the service through EIP `1.1.1.1`.

![](https://pek3b.qingstor.com/kubesphere-docs/png/20200411233327.png)

## Deployment Architecture

Porter serves as a Load-Balancer plugin, monitoring the changes of the service in the cluster through a `Manager`, and advertises related routes. At the same time, all the nodes in the cluster are deployed with an agent. Each time an EIP is used, a host routing rule will be added to the host, diverting the IP packets sent to the EIP to the local.

![](https://pek3b.qingstor.com/kubesphere-docs/png/20200411233355.png)

## Logic

When Porter is deployed as a service in Kubernetes cluster, it establishes a BGP connection with the cluster's border router (Layer 3 switch). When a service with a specific annotation (e.g. `lb.kubesphere.io/v1apha1: porter`, see [Example Service](https://github.com/kubesphere/porter/blob/master/config/samples/service.yaml)) has been created in the cluster, the service is dynamically assigned an EIP (users can also specify EIP by themselves). The LB controller creates a route and forwards the route to the public network (or private network) through BGP, so that the service can be accessed externally.

The Porter LB controller is a custom controller based on the [Kubernetes controller runtime](https://github.com/kubernetes-sigs/controller-runtime), which automatically update the routing information by watching changes of the service.

![](https://pek3b.qingstor.com/kubesphere-docs/png/20200411234522.png)
