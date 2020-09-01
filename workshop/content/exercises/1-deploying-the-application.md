We will use a small [Go application](https://github.com/eduk8s-labs/sample-app-go) as our example to showcase how tools from Carvel can work together to aid in the process of developing and deploying an application.

A pre-built container image for the application is published at `quay.io/eduk8s-labs/sample-app-go:latest`.

The current directory contains multiple sub directories, with variations of our application configuration, that we will use during the workshop. To view the files run:

```execute
tree .
```

The output should be:

```
.
├── config-step-1-minimal
│   └── config.yml
├── config-step-2a-overlays
│   └── custom-scale.yml
├── config-step-2b-multiple-data-values
│   └── values.yml
├── config-step-2-template
│   ├── config.yml
│   └── values.yml
├── config-step-3-build-local
│   ├── build.yml
│   ├── config.yml
│   └── values.yml
└── config-step-4-build-and-push
    ├── build.yml
    ├── config.yml
    ├── push-secret.yml
    ├── push.yml
    └── values.yml

6 directories, 13 files
```

When deploying the container image for an application to Kubernetes, you will typical use a Kubernetes deployment resource. In order to expose the application under a single IP address within the Kubernetes cluster, or via an external load balancer or ingress, you will also use a service resource.

In our first example, this is what the `config-step-1-minimal/config.yml` file contains.

To view the contents of the file run:

```execute
cat config-step-1-minimal/config.yml
```

As is the case with self contained resource configurations for Kubernetes, the name of the container image and the environment variable `HELLO_MSG` are hard coded into the deployment resource.

If using `kubectl` to deploy the application to Kubernetes, you would use the command:

```
kubectl apply -f config-step-1-minimal/config.yml
```

We will not be using `kubectl`, but to see what it would output if run, you can run it with the `--dry-run=client` option.

```execute
kubectl apply -f config-step-1-minimal/config.yml --dry-run=client
```

All that `kubectl` would output is the list of affected resources.

```
deployment.apps/simple-app created
service/simple-app created
```

What `kubectl` does not do is tell you the specifics of the changes that are being made so you can confirm that is what you expected.

Further, `kubectl` does not yet have a robust pruning capability to converge a set of resources ([GitHub issue](https://github.com/kubernetes/kubectl/issues/572)) when configuration is re-applied to an existing set of deployment resources.

The [kapp](https://get-kapp.io/) tool from Carvel addresses and improves on these and other limitations of `kubectl`, as it is designed around the concept of a Kubernetes application, that being, a set of related resources pertaining to an application, which share a common label.

Key principles around which `kapp` is designed are:

* `kapp` separates change calculation phase (diff), from change apply phase (apply) to give users visibility and confidence regarding what's about to change in the cluster.
* `kapp` tracks and converges resources based on a unique generated label, freeing its users from worrying about cleaning up old deleted resources as the application is updated.
* `kapp` orders resources when being applied so that the Kubernetes API server can successfully process them when dependencies exist (e.g., CRDs and namespaces are created before other resources).
* `kapp` tries to wait for resources to become ready before considering the deployment a success.

To deploy our application with `kapp`, run:

```execute
kapp deploy -a simple-app -f config-step-1-minimal/
```

You will be prompted to accept the changes before they are applied. Enter `y` when asked to continue.

```terminal:input
text: y
```

The output from `kapp` should be similar to the following:

```
Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
{{session_namespace}}  simple-app  Deployment  -       -    create  reconcile  -   -
^                                  simple-app  Service     -       -    create  reconcile  -   -

Op:      2 create, 0 delete, 0 update, 0 noop
Wait to: 2 reconcile, 0 delete, 0 noop

Continue? [yN]: y

11:17:38AM: ---- applying 2 changes [0/2 done] ----
11:17:38AM: create service/simple-app (v1) namespace: {{session_namespace}}
11:17:38AM: create deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:17:38AM: ---- waiting on 2 changes [0/2 done] ----
11:17:38AM: ok: reconcile service/simple-app (v1) namespace: {{session_namespace}}
11:17:39AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:17:39AM:  ^ Waiting for 1 unavailable replicas
11:17:39AM:  L ok: waiting on replicaset/simple-app-6b4488bc45 (apps/v1) namespace: {{session_namespace}}
11:17:39AM:  L ongoing: waiting on pod/simple-app-6b4488bc45-5mmfr (v1) namespace: {{session_namespace}}
11:17:39AM:     ^ Pending: ContainerCreating
11:17:39AM: ---- waiting on 1 changes [1/2 done] ----
11:17:41AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:17:41AM:  ^ Waiting for 1 unavailable replicas
11:17:41AM:  L ok: waiting on replicaset/simple-app-6b4488bc45 (apps/v1) namespace: {{session_namespace}}
11:17:41AM:  L ok: waiting on pod/simple-app-6b4488bc45-5mmfr (v1) namespace: {{session_namespace}}
11:17:43AM: ok: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:17:43AM: ---- applying complete [2/2 done] ----
11:17:43AM: ---- waiting complete [2/2 done] ----

Succeeded
```

To list the applications that have been deployed to the current namespace, you can run:

```execute-1
kapp ls
```

This will list a single application called `simple-app`, corresponding to the name of the application we specified using the `-a` option when `kapp` was originally run to deploy the application.

```
Apps in namespace '{{session_namespace}}'

Name        Namespaces                         Lcs   Lca
simple-app  {{session_namespace}}  true  30s

Lcs: Last Change Successful
Lca: Last Change Age

1 apps

Succeeded
```

To inspect all the Kubernetes resources created for the application called `sample-app`, run:

```execute
kapp inspect -a simple-app --tree
```

The output should be similar to:

```
Resources in app 'simple-app'

Namespace                          Name                             Kind        Owner    Conds.  Rs  Ri  Age
{{session_namespace}}  simple-app                       Service     kapp     -       ok  -   1m
{{session_namespace}}   L simple-app                    Endpoints   cluster  -       ok  -   1m
{{session_namespace}}  simple-app                       Deployment  kapp     2/2 t   ok  -   1m
{{session_namespace}}   L simple-app-9b466965b          ReplicaSet  cluster  -       ok  -   1m
{{session_namespace}}   L.. simple-app-9b466965b-jzpzs  Pod         cluster  4/4 t   ok  -   1m

Rs: Reconcile state
Ri: Reconcile information

5 resources

Succeeded
```

As you can see, `kapp` will list resources it did not directly create (such as replicaset, pods and endpoints), but which are created as a side effect of the deployment and service resources being created.

You can also get the logs for the application:

```execute
kapp logs -f -a simple-app
```

For our simple Go application you should see log output similar to:

```
# starting tailing 'simple-app-6f884d8d9d-nn5ds > simple-app' logs
simple-app-6f884d8d9d-nn5ds > simple-app | 2019/05/09 20:43:36 Server started
```

The `inspect` and `logs` commands make it convenient to view resources related to an application by using only the name of the application.

For example, the `logs` command in this case (since we use `-f`) will tail any existing or new pod that is part of `simple-app` application, even after we make changes and redeploy, without needing to identify the names of the individual pods.
