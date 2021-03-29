---
title: "Build the PorterLB Project"
linkTitle: "Build the PorterLB Project"
weight: 1
---

This document describes how to build the PorterLB project for testing.

## Prerequisites

* You need to prepare a Linux environment.
* You need to install [Go 1.12 or later](https://github.com/kubesphere/porter/blob/master/doc/how-to-build.md).
* You need to install [Docker](https://www.docker.com/get-started).
* You need to install [Docker Buildx](https://www.docker.com/blog/getting-started-with-docker-for-arm-on-linux/).

## Procedure

1. Visit https://github.com/kubesphere/porterlb and click **Fork** to fork the PorterLB repository to your own GitHub account.

2. Log in to your environment, and run the following commands to clone the PorterLB repository and go to the `porter` directory:

   ```bash
   git clone <Address of your own PorterLB repository>
   ```

   ```bash
   cd porter
   ```

3. Run the following command to install Kustomize and Kubebuilder:

   ```bash
   ./hack/install_tools.sh
   ```

4. Run the following command to install controller-gen:

   ```bash
   go get sigs.k8s.io/controller-tools/cmd/controller-gen@v0.4.0
   ```

5. Run the following command to configure the environment variable for controller-gen:

   ```bash
   export PATH=/root/go/bin/:$PATH
   ```

   {{< notice note >}}

   You need to change `/root/go/bin/` to the actual path of controller-gen.

   {{</ notice >}}

6. Run the following command to generate CRDs and webhooks:

   ```bash
   make generate
   ```

7. Customize the values of `IMG_MANAGER` and `IMG_AGENT` in `Makefile` and run the following command to generate a YAML release file in the `deploy` directory:

   ```bash
   make release
   ```

   {{< notice note >}}

   * `IMG_MANAGER` specifies the repository and tag of the porter-manager image.

   * `IMG_AGENT` specifies the repository and tag of the porter-agent image.
   * Currently, PorterLB uses only the porter-manager image. The porter-agent image will be used in future versions.

   {{</ notice >}}

8. Run the following command to deploy PorterLB as a plugin:

   ```bash
   kubectl apply -f deploy/release.yaml
   ```

   