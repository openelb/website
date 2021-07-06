---
title: "Overview"
linkTitle: "Overview"
weight: 1
---

OpenELB is an open-source load balancer implementation designed for bare-metal Kubernetes clusters.

## Why OpenELB

In cloud-based Kubernetes clusters, Services are usually exposed by using load balancers provided by cloud vendors. However, cloud-based load balancers are unavailable in bare-metal environments. OpenELB allows users to create LoadBalancer Services in bare-metal, edge, and virtualization environments for external access, and provides the same user experience as cloud-based load balancers.

## Core Features

- BGP mode and Layer 2 mode
- ECMP routing and load balancing
- IP address pool management
- BGP configuration using CRDs
- Installation using Helm Chart

## Support, Discussion and Contributing

OpenELB is a sub-project of [KubeSphere](https://github.com/kubesphere).

* Join us at the [KubeSphere Slack Channel](https://kubesphere.slack.com/join/shared_invite/enQtNTE3MDIxNzUxNzQ0LTZkNTdkYWNiYTVkMTM5ZThhODY1MjAyZmVlYWEwZmQ3ODQ1NmM1MGVkNWEzZTRhNzk0MzM5MmY4NDc3ZWVhMjE#/) to get support or simply tell us that you are using OpenELB.
* You have code or documents for OpenELB? Contributions are always welcome! See [Building and Contributing](/docs/building-and-contributing/) to obtain guidance.

## License

OpenELB is licensed under the Apache License, Version 2.0. See [LICENSE](https://github.com/kubesphere/porter/blob/master/LICENSE) for the full license text.