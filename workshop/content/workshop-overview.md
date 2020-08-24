[Carvel](https://carvel.dev/) contains several tools we created as a result of working with complex, multi-purpose tools like [Helm](https://helm.sh/). We believe that working with simple, single-purpose tools that easily interoperate with one another results in a better, workflow compared to the all-in-one approach chosen by Helm. We have found this approach to be easier to understand and debug.

In this workshop we will focus on local application development workflow; however, tools introduced here work also well for other workflows, for example, for production GitOps deployments or manual application deploys. We plan to publish additional blog posts for other workflows. Let us know what you are most interested in!

We break down local application development workflow into the following stages:

1. Source code authoring
1. Configuration authoring (e.g. YAML configuration files)
1. Packaging (e.g. Dockerfile)
1. Deployment (e.g. kubectl apply ...)
1. Repeat!

Helm arguably tries to address stages 2, 3, and 4, with configuration, packaging and deployment together in one tool. The community has varied opinions on [advantages](https://medium.com/@aevitas/drastically-improve-your-kubernetes-deployments-with-helm-5323e7f11ef8) and [disadvantages](https://medium.com/@slynko/experiences-with-upgrading-using-helm-b23dc0ca683d?_branch_match_id=494645732166043546) of using Helm. However, let's explore an alternative approach with Carvel.

For each stage, we have open sourced a tool that we believe addresses that stage's challenges (sections below explore each tool in detail):

* __configuration__ -> [ytt](https://get-ytt.io/) for YAML configuration and templating
* __packaging__ -> [kbld](https://get-kbld.io/) for building Docker images and record image references
* __deployment__ -> [kapp](https://get-kapp.io/) for deploying k8s resources
