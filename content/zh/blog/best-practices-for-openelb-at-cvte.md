---
title: "OpenELB 在 CVTE 的最佳实践"
linkTitle: "OpenELB 在 CVTE 的最佳实践"
description: 视源股份是怎么使用 OpenELB 的？
keywords: OpenELB, Bare Metal, Kubernetes, KubeSphere, Load Balancer, Cloud Native
author: 裴振飞
date: 2023-06-14
weight: 100000
---

## 公司介绍

广州视源电子科技股份有限公司（以下简称视源股份）成立于 2005 年 12 月，旗下拥有多家业务子公司。截至 2022 年 12 月 31 日，公司总人数超 6000 人，约 60% 为技术人员，员工平均年龄约为 29 岁。

目前公司的主营业务为液晶显示主控板卡和交互智能平板等显控产品的设计、研发与销售，产品已广泛应用于家电领域、 教育信息化领域、企业服务领域等，始终致力于通过产品创新、研发设计提升产品的用户体验，为客户和用户持续创造价值。公司自成立以来，依托在音视频技术、信号处理、电源管理、人机交互、应用开发、系统集成等电子产品领域的软硬件技术积累，面向多应用场景进行技术创新和产品开发，通过产品和资源整合等能力在细分市场逐步取得领先地位，并建立了教育数字化工具及服务提供商希沃（seewo）、智慧协同平台 MAXHUB 等多个业内知名品牌。

## 挑战

随着在 Kubernetes 上运行的应用程序数量增加，如何优雅地将服务导出到集群成为一个需要解决的问题。Kubernetes 提供了 NodePort 和 Loadbalancer 来导出服务。NodePort 受到主机端口数量的限制，过多的端口开放还会有安全风险，在流量治理方面更是让人望而生畏；Loadbalancer 则用于声明云供应商的服务，对于裸金属服务器而言，Kubernetes 官方不提供支持。对于企业私有云中的 Kubernetes 集群，想要对外提供服务，就遇到了挺大的挑战。在这种情况下，我们发现了 OpenELB, 它可以在 Loadbalancer 类型中导出服务并稳定运行。

## 解决方案

作为 KubeSphere 团队开源的项目，OpenELB（曾用名 Porter）是一款为裸金属服务器提供 LoadBalancer 类型服务的产品。

总结起来，有如下几个优点：

- 首先，它是由一个团队而不是个人运行的，而且社区贡献者颇多，因此可以避免厂商依赖或者无人维护。
- 其次，OpenELB 支持的协议比同类竞争对手更多。
- 最后，OpenELB 有完备的文档，对中文支持友好，企业服务可以获得更多的技术服务支持。在使用过程中，您会发现更多的惊喜。

## 实践过程

管理员可以通过 NodePort 导出 Kubernetes 服务。在这种情况下，每台集群主机都会导出同样的端口，这在大规模集群中的开销是巨大的。不幸的是，Kubernetes 可以使用的主机端口是有限的（30000-32768）。开源社区建议使用动态端口，这可能会带来风险和管理难度，给流量治理带来巨大的挑战。Kubernetes 中的默认 LoadBalancer 类型的服务仅支持云厂商提供的服务。

![](http://pek3b.qingstor.com/kubesphere-community/images/k8s_NodePort_Model.excalidraw.png)

OpenELB 是一种解决方案，用于为裸机提供负载均衡类型的服务，基于 BGP 和 ECMP 协议，可实现最佳性能和高可用性。OpenELB 提供了两个功能模块：

1. 负载均衡控制器（Controller）

   该控制器负责将 BGP 路由同步到物理交换机。

2. CRD 和 CRD 控制器

   CRD EIP 定义了在 Layer2 模式中使用的 EIP 地址池，同样，CRD BgpConf 和 BgpPeer 定义了 Kubernetes 集群内部的 BGP，并用于 BGP 模式。

![](http://pek3b.qingstor.com/kubesphere-community/images/OpenELB-Infrastructure.excalidraw.png)


生产环境中，强烈建议使用 BGP 模式。如下图所示，把物理路由器设备划归到 AS50001 中，把 OpenELB 所在的集群划归到 AS 50000 中，OpenELB 会建立服务路由规则，并将其同步到物理路由器上，BGP 路由器根据负载均衡策略转发对应的流量到达相应的集群主机节点上，kube-proxy 再近一步转发流量到 Service 层。BGP 模式需要物理路由器设备和集群主机网络的支持，其最大的优点就是稳定可靠，解决了路由转发的单点故障问题。

![](https://openelb.io/images/en/docs/concepts/vip-mode-beta/openelb-vip-mode-topology.png)


虽然 BGP 模式是建议的模式，但是在实际应用中，Layer2 模式使用更为广泛。因为它简单易用，配置要求低，配置简单。对于使用虚拟机作为 K8s 集群节点的用户，受 VMware 网络的限制，只能选择 Layer2 模式。让我们来看下 Layer2 模式的网络架构。

我们使用 GitLab CI 和 Argo CD 建立自动化流水线，以帮助开发人员提高效率，并使用 Nginx Ingress 处理 7 层请求。Argo CD 使用 IP `10.10.1.2` 导出负载均衡类型的服务，而 Nginx Ingress 则使用 IP `10.10.1.3` 导出。因此，我们可以建立如下的 DNS 记录：

```text
# HOSTS
10.10.1.2  argocd.cvte.com
10.10.1.3  ingress.cvte.com
```

当用户访问像 `argocd.cvte.com` 这样的域名时，请求将路由到路由器。由于 EIP 不能绑定到任何网络卡硬件上，因此路由器需要广播 ARP 请求，这个请求将被 OpenELB 回应一个集群节点的 IP。这样，路由器就知道将网络数据包发送到哪里了。由 Ingress 控制的任何其他域都会路由到另一个 EIP。

通常情况下，OpenELB 仅会使用一个集群节点 IP 响应 ARP 请求。如果此节点关闭或不可用，则 OpenELB 会切换到另一个节点，以确保业务的连续性。

![](http://pek3b.qingstor.com/kubesphere-community/images/CVTE-OpenELB.excalidraw.png)

通过对 Layer2 模式原理的讲解，不难发现，Layer2 模式有单点故障风险。为了减少这种单点风险，KubeSphere 研发团队贴心地开发了 VIP 模式，即虚拟 IP 模式。VIP 提供了单点故障的转移，当 Layer2 模式中的主机节点故障，OpenELB 会自动切换流量到备份 IP。VIP 模式，只能算是减少单点故障风险，无法完全消灭风险。所以在生产环境中，普通 Layer2 模式或 VIP 模式，都不建议使用。根据我们的经验，如万般无奈，只能使用 Layer2 模式，那就独立两台机器，只用来接收外部流量，并尽量保证这两台机器的高可用。

![](https://openelb.io/images/en/docs/concepts/layer-2-mode/openelb-layer-2-topology.png)

## 使用效果

OpenELB 以 TCP 协议导出服务，并与 Ingress 配合使用，可以轻松管理大多数 Web 应用程序。与 NodePort 不同，OpenELB 只需要几个 EIP 即可代理请求到服务，流量集中处理从而极大地降低了管理难度。

## 总结

OpenELB 提供了一种为裸金属服务器导出 LoadBalancer 类型服务的功能，帮助我们轻松地管理网络基础设施。作为 KubeSphere 的一个子项目，OpenELB 非常适合在企业私有云环境中使用，具有较强的稳定性和可信度。