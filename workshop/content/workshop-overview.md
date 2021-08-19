# Overview

[Carvel](https://carvel.dev/) contains several tools created out of the experience of having to work with complex, multi-purpose tools like [Helm](https://helm.sh/). Working with simple, single-purpose tools, that easily interoperate with one another can result in a better workflow compared to the all-in-one approach chosen by Helm. Using this approach can be easier to understand and debug.

In this workshop we will focus on the local application development workflow; however, the tools from Carvel introduced here can also work well for other workflows, for example, for production GitOps deployments or manual application deployments.

A local application development workflow can be broken down into the following stages:

1. Source code authoring
1. Configuration authoring (e.g. YAML configuration files)
1. Packaging (e.g. Dockerfile)
1. Deployment (e.g. kubectl apply ...)
1. Repeat!

Helm arguably tries to address stages 2, 3, and 4, with configuration, packaging and deployment together in one tool. The Kubernetes community has varied opinions on the [advantages](https://medium.com/@aevitas/drastically-improve-your-kubernetes-deployments-with-helm-5323e7f11ef8) and [disadvantages](https://medium.com/@slynko/experiences-with-upgrading-using-helm-b23dc0ca683d?_branch_match_id=494645732166043546) of using Helm. This workshop therefore aims to show you an alternative approach using tools from Carvel.

For each stage, Carvel provides a tool that aims to address that stage's challenges:

* __sourcing__ -> [vendir](https://carvel.dev/vendir/) syncs any number of data sources (e.g., Git repositories) into a consistent structure consuming a YAML definition
* __configuration__ -> [ytt](https://carvel.dev/ytt/) for YAML configuration and templating
* __packaging__ -> [kbld](https://carvel.dev/kbld/) for building container images and recording image references and [imgpkg](https://carvel.dev/imgpkg/) to package, distribute, and relocate Kubernetes configuration and dependent OCI images as one OCI artifact
* __deployment__ -> [kapp](https://carvel.dev/kapp/) for deploying a set of Kubernetes resources

Bringing all the above together and thinking about how authorship and consumption of sets of versioned Kubernetes resources we will take a look at [kapp-controller](https://carvel.dev/kapp/) where:

 * Package authors may encapsulate, version, and distribute software for others to install on a Kubernetes cluster
 * Package consumers may discover, configure, and install versioned software on a Kubernetes cluster without needing to know all the underlying details

The kapp-controller will continuously fetch, template, and deploy an installed package just like an application.