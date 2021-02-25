---
title: "An Open-Source Load Balancer for Bare-Metal Kubernetes"
linkTitle: "KubeCon Sharing"
date: 2019-06-24
description: >
  The video to introduce the PorterLB project in KubeCon.
weight: 100001
---

As we know, the backend workload can be exposed externally using service of type "LoadBalancer" in Kubernetes cluster. Cloud vendors often provide cloud LB plugins for Kubernetes which requires the cluster to be deployed on a specific IaaS platform. However, many enterprise users usually deploy Kubernetes clusters on bare meta especially for production use. For the on-premise bare meta clusters, Kubernetes does not provide Load-Balancer implementation. PorterLB, an open-source project, is the right solution for such issue.  

This video will focus on the network technologies to help expose service and EIP management for bare meta Kubernetes cluster.

<iframe width="560" height="315" src="https://www.youtube.com/embed/EjU1yAVxXYQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
