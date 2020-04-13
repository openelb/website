---
title: "Overview"
linkTitle: "Overview"
weight: 1
description: >
  What is Porter
---

{{% pageinfo %}}
This guide walk you through the basic introduction of Porter.
{{% /pageinfo %}}

## What is Porter

[Porter](https://github.com/kubesphere/porter) is an open source load balancer designed for bare metal Kubernetes clusters. It's implemented by physical switch, and uses BGP and ECMP to achieve the best performance and high availability.

## Why We Need Porter

As we know, In the cloud-hosted Kubernetes cluster, the cloud providers (AWS, GCP, Azure, etc.) usually provide the Load Balancer to assign IPs and expose the service to outside.

However, the service is hard to expose in a bare metal cluster since Kubernetes does not provide a load-balancer for bare metal environment. Fortunately, Porter allows you to create Kubernetes services of type “LoadBalancer” in bare metal clusters, which makes you have consistent experience with the cloud.

## Core Features

Porter has two components which provide the following core features:

- LB Controller & Agent: The controller is responsible for synchronizing BGP routes to the physical switch; The agent is deployed to each node as DaemonSet to maintain the drainage rules;

- EIP service, including the EIP pool management and EIP controller, the controller is responsible for dynamically updating the EIP information of the service.

## Integrate with KubeSphere

Porter is a subproject of [KubeSphere Container Platform](https://github.com/kubesphere/kubesphere), it allows users to integrate with KubeSphere seamlessly.
