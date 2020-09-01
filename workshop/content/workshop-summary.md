We've seen how [ytt](https://get-ytt.io/), [kbld](https://get-kbld.io/) and [kapp](https://get-kapp.io/) can be used together (`ytt ... | kbld -f- | kapp deploy ...`) to deploy and iterate on an application running on Kubernetes. Each one of these tools has been designed to be single-purpose and composable with other tools from Carvel and the larger Kubernetes ecosystem.

The Carvel development team is eager to hear your thoughts and feedback in [#k14s in Kubernetes slack](https://slack.kubernetes.io/) and/or via Github issues and PRs ([https://github.com/k14s/](https://github.com/k14s/)) for each project. Don't hesitate to reach out!

## Authors

This workshop was based on a [blog post](https://tanzu.vmware.com/content/blog/introducing-k14s-kubernetes-tools-simple-and-composable-tools-for-application-deployment) written by:

* __Dmitriy Kalinin__ is a Software Engineer at Pivotal working on Kubernetes and Cloud Foundry projects. ([@dmitriykalinin](https://twitter.com/dmitriykalinin) on twitter)
* __Nima Kaviani__ is a Senior Cloud Engineer with IBM. Nima has been a contributor to Cloud Foundry, Kubernetes, and Knative open source projects. He holds a PhD in Computer Science and tweets and blogs about distributed systems, life, and technology in general. ([@nimak](https://twitter.com/nimak) on twitter)
