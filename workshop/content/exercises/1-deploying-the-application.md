# Deploying an application

We will use a small [Go application](https://github.com/eduk8s-labs/sample-app-go) as our example to showcase how tools from Carvel can work together to aid in the process of developing and deploying an application.

A pre-built container image for the application is published at [quay.io/eduk8s-labs/sample-app-go:latest](https://quay.io/repository/eduk8s-labs/sample-app-go?tag=latest&tab=tags). We will use this initially, but in later steps we will use one of the Carvel tools to coordinate the building of the container image from source code.

The current directory contains multiple sub directories with variations of our application configuration that we will use during the workshop.

To view the files, run:

```
tree .
```

The output should be something like:

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

When deploying the container image for an application to Kubernetes, you will typically use a Kubernetes deployment resource. In order to expose the application under a single IP address within the Kubernetes cluster, or via an external load balancer or ingress, you will also use a service resource.

In our first example, this is what the `config-step-1-minimal/config.yml` file contains.

To view the contents of the file, run:

```
cat config-step-1-minimal/config.yml
```

As is the case with self contained resource configurations for Kubernetes, the name of the container image and an environment variable `HELLO_MSG` needed by the application are hard coded into the deployment resource.

If using `kubectl` to deploy the application to Kubernetes, you would use the command:

```
kubectl apply -f config-step-1-minimal/config.yml
```

We will not be using `kubectl`, but to see what it would output if run, you can run it with the `--dry-run=client` option.

```
kubectl apply -f config-step-1-minimal/config.yml --dry-run=client -o yaml
```

All that `kubectl` would output is the list of affected resources.

```
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"simple-app","namespace":"default"},"spec":{"selector":{"matchLabels":{"simple-app":""}},"template":{"metadata":{"labels":{"simple-app":""}},"spec":{"containers":[{"env":[{"name":"HELLO_MSG","value":"stranger"}],"image":"quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296","name":"simple-app"}]}}}}
    name: simple-app
    namespace: default
  spec:
    selector:
      matchLabels:
        simple-app: ""
    template:
      metadata:
        labels:
          simple-app: ""
      spec:
        containers:
        - env:
          - name: HELLO_MSG
            value: stranger
          image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
          name: simple-app
- apiVersion: v1
  kind: Service
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"simple-app","namespace":"default"},"spec":{"ports":[{"port":8080,"targetPort":8080}],"selector":{"simple-app":""}}}
    name: simple-app
    namespace: default
  spec:
    ports:
    - port: 8080
      targetPort: 8080
    selector:
      simple-app: ""
kind: List
metadata: {}
```


Rather unfortunately `kubectl` does not yet have a robust pruning capability to converge a set of resources ([GitHub issue](https://github.com/kubernetes/kubectl/issues/572)) when configuration is re-applied to an existing set of deployment resources.

The [kapp](https://carvel.dev/kapp/) tool from Carvel addresses and improves on this and other limitations of `kubectl`, as it is designed around the concept of a Kubernetes application, that being, a set of related resources pertaining to an application, which share a common label.

Key principles around which `kapp` is designed are:

* `kapp` separates the change calculation phase (diff) from the change apply phase (apply) to give users visibility and confidence regarding what's about to change in the cluster.
* `kapp` tracks and converges resources based on a unique generated label, freeing its users from worrying about cleaning up old deleted resources as the application is updated.
* `kapp` orders resources when being applied so that the Kubernetes API server can successfully process them when dependencies exist (e.g., CRDs and namespaces are created before other resources).
* `kapp` tries to wait for resources to become ready before considering the deployment a success.

To deploy our application with `kapp`, run:

```
kapp deploy -a simple-app -f config-step-1-minimal/
```
> The `-a` argument above is the name we wish to associate with the application (resources) we wish to deploy.

You will be prompted to accept the changes before they are applied. Enter `y` when asked to continue.

The output from `kapp` should be similar to the following:

```
Changes

Namespace  Name        Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
default    simple-app  Deployment  -       -    create  -       reconcile  -   -
^          simple-app  Service     -       -    create  -       reconcile  -   -

Op:      2 create, 0 delete, 0 update, 0 noop
Wait to: 2 reconcile, 0 delete, 0 noop

Continue? [yN]: y

11:29:52PM: ---- applying 2 changes [0/2 done] ----
11:29:52PM: create deployment/simple-app (apps/v1) namespace: default
11:29:52PM: create service/simple-app (v1) namespace: default
11:29:52PM: ---- waiting on 2 changes [0/2 done] ----
11:29:52PM: ok: reconcile service/simple-app (v1) namespace: default
11:29:52PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
11:29:52PM:  ^ Waiting for generation 2 to be observed
11:29:52PM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
11:29:52PM:  L ongoing: waiting on pod/simple-app-677b96597b-qhjjm (v1) namespace: default
11:29:52PM:     ^ Pending
11:29:52PM: ---- waiting on 1 changes [1/2 done] ----
11:29:52PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
11:29:52PM:  ^ Waiting for 1 unavailable replicas
11:29:52PM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
11:29:52PM:  L ongoing: waiting on pod/simple-app-677b96597b-qhjjm (v1) namespace: default
11:29:52PM:     ^ Pending: ContainerCreating
11:29:55PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
11:29:55PM:  ^ Waiting for 1 unavailable replicas
11:29:55PM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
11:29:55PM:  L ok: waiting on pod/simple-app-677b96597b-qhjjm (v1) namespace: default
11:29:56PM: ok: reconcile deployment/simple-app (apps/v1) namespace: default
11:29:56PM: ---- applying complete [2/2 done] ----
11:29:56PM: ---- waiting complete [2/2 done] ----

Succeeded
```

To list the applications that have been deployed to the current namespace, you can run:

```
kapp ls
```

This will list all applications deployed, among which is `simple-app`

```
Apps in namespace 'default'

Name          Namespaces                             Lcs   Lca
cert-manager  (cluster),cert-manager                 true  52m
kc            (cluster),kapp-controller,kube-system  true  52m
simple-app    default                                true  1m

Lcs: Last Change Successful
Lca: Last Change Age

3 apps

Succeeded
```

To inspect all the Kubernetes resources created for the application called `sample-app`, run:

```
kapp inspect -a simple-app --tree
```

The output should be similar to:

```
Resources in app 'simple-app'

Namespace  Name                              Kind           Owner    Conds.  Rs  Ri  Age
default    simple-app                        Deployment     kapp     2/2 t   ok  -   5m
default     L simple-app-677b96597b          ReplicaSet     cluster  -       ok  -   5m
default     L.. simple-app-677b96597b-qhjjm  Pod            cluster  4/4 t   ok  -   5m
default    simple-app                        Service        kapp     -       ok  -   5m
default     L simple-app                     Endpoints      cluster  -       ok  -   5m
default     L simple-app-drqd7               EndpointSlice  cluster  -       ok  -   5m

Rs: Reconcile state
Ri: Reconcile information

6 resources

Succeeded
```

As you can see, `kapp` will also list resources it did not directly create (such as replicaset, pods and endpoints), but which are created as a side effect of the deployment and service resources being created.

You can also get the logs for the application:

```
kapp logs -a simple-app
```

For our simple Go application you should see log output similar to:

```
# starting tailing 'simple-app-677b96597b-qhjjm > simple-app' logs
simple-app-677b96597b-qhjjm > simple-app | 2021/08/18 23:29:55 Server started
# ending tailing 'simple-app-677b96597b-qhjjm > simple-app' logs

Succeeded
```

The `inspect` and `logs` commands make it convenient to view resources related to an application by using only the name of the application.

We could have added a `-f` before the application to tail any existing or new pod that is part of `simple-app` application, even after we make changes and redeploy, without needing to identify the names of the individual pods.

To illustrate this, kill the currently running pod using:

```
kubectl delete pods -l simple-app
```

then type

```
kapp logs -a simple-app
```

Note the new replacement pod name

```
# starting tailing 'simple-app-677b96597b-d8vfz > simple-app' logs
simple-app-677b96597b-d8vfz > simple-app | 2021/08/18 23:41:50 Server started
# ending tailing 'simple-app-677b96597b-d8vfz > simple-app' logs

Succeeded
```

If you had another terminal window opened and had used `kapp logs -a -f simple-app`, you would have seen logging from the original pod stopping, and the logging from the replacement pod starting.

If you were using `kubectl logs -f deployment/simple-app`, killing of the pod would have caused `kubectl logs` to exit.
