We'll use [sample-app-go](https://github.com/eduk8s-labs/sample-app-go) application as our example to showcase how these tools can work together to develop and deploy an application. 

A working image for the application is published at `quay.io/eduk8s-labs/sample-app-go:latest`

Our current directory (`exercises`) contains multiple `config-step-*` directories with variations of application configuration that we will use during the workshop.

```execute-1
ls -la
```

```
[~/exercises] $ ls -la
total 0
drwxrwxr-x 1 eduk8s root 208 May 27 15:17 .
drwxrwxr-x 1 eduk8s root  90 May 27 17:24 ..
drwxrwxr-x 1 eduk8s root  24 May 27 12:25 config-step-1-minimal
drwxrwxr-x 1 eduk8s root  30 May 27 12:25 config-step-2a-overlays
drwxrwxr-x 1 eduk8s root  24 May 27 12:25 config-step-2b-multiple-data-values
drwxrwxr-x 1 eduk8s root  42 May 27 12:25 config-step-2-template
drwxrwxr-x 1 eduk8s root  59 May 27 12:25 config-step-3-build-local
drwxrwxr-x 1 eduk8s root  75 May 27 12:25 config-step-4-build-and-push
```

Typically, an application deployed to Kubernetes will include Deployment and Service resources in its configuration. In our example, `config-step-1-minimal/` directory contains `config.yml` which contains exactly that. (Note that the Docker image is already preset and environment variable `HELLO_MSG` is hard coded. We'll get to those shortly.)

Traditionally, you can use `kubectl apply -f config-step-1-minimal/config.yml` to deploy this application. However, kubectl (1) does not indicate which resources are affected and how they are affected before applying changes, and (2) does not yet have a robust prune functionality to converge a set of resources ([GH issue](https://github.com/kubernetes/kubectl/issues/572)). __[kapp](https://get-kapp.io/)__ addresses and improves on several kubectl's limitations as it was designed from the start around the notion of a "__Kubernetes Application__" - `a set of resources with the same label`:

* kapp separates change calculation phase (diff), from change apply phase (apply) to give users visibility and confidence regarding what's about to change in the cluster
* kapp tracks and converges resources based on a unique generated label, freeing its users from worrying about cleaning up old deleted resources as the application is updated
* kapp orders certain resources so that the Kubernetes API server can successfully process them (e.g., CRDs and namespaces before other resources)
* kapp tries to wait for resources to become ready before considering the deploy a success

Let us deploy our application with kapp:

```execute-1
kapp deploy -a simple-app -f config-step-1-minimal/
```

You should see a similar output to the following:
```
Changes

Namespace  Name        Kind        Conditions  Age  Changed  Ignored Reason
default    simple-app  Deployment  -           -    add      -
~          simple-app  Service     -           -    add      -

2 add, 0 delete, 0 update, 0 keep

2 changes

Continue? [yN]: y

6:11:44PM: --- applying changes
6:11:44PM: add service/simple-app (v1) namespace: default
6:11:44PM: add deployment/simple-app (apps/v1) namespace: default
6:11:44PM: waiting on add service/simple-app (v1) namespace: default
6:11:44PM:  L waiting on endpoints/simple-app (v1) namespace: default ... done
6:11:44PM: waiting on add deployment/simple-app (apps/v1) namespace: default
6:11:44PM:  L waiting on replicaset/simple-app-5fb5ff9bdb (extensions/v1beta1) namespace: default ... done
6:11:44PM:  L waiting on pod/simple-app-5fb5ff9bdb-2clpj (v1) namespace: default ... in progress: Pending
6:11:45PM:
6:11:45PM: ---  waiting on 1 changes
6:11:45PM: waiting on add deployment/simple-app (apps/v1) namespace: default
6:11:45PM:  L waiting on replicaset/simple-app-5fb5ff9bdb (apps/v1) namespace: default ... done
6:11:45PM:  L waiting on pod/simple-app-5fb5ff9bdb-2clpj (v1) namespace: default ... done
6:11:45PM: --- changes applied

Succeeded
```

__TODO__: This label seems not to exist anymore
Our `simple-app` received a unique label `kapp.k14s.io/app=1557433075084066000` for resource tracking:

```execute-1
kapp ls
```

```
Apps in namespace 'lab-getting-started-k14s-w01-s001'

Name        Namespaces                         Lcs   Lca
simple-app  lab-getting-started-k14s-w01-s001  true  21s

Lcs: Last Change Successful
Lca: Last Change Age

1 apps

Succeeded
```

Using this label, kapp tracks and allows inspection of all Kubernetes resources created for `sample-app`:

```execute-1
kapp inspect -a simple-app --tree
```

```
Resources in app 'simple-app'

Namespace                          Name                             Kind        Owner    Conds.  Rs  Ri  Age
lab-getting-started-k14s-w01-s001  simple-app                       Service     kapp     -       ok  -   7m
lab-getting-started-k14s-w01-s001   L simple-app                    Endpoints   cluster  -       ok  -   7m
lab-getting-started-k14s-w01-s001  simple-app                       Deployment  kapp     2/2 t   ok  -   7m
lab-getting-started-k14s-w01-s001   L simple-app-9b466965b          ReplicaSet  cluster  -       ok  -   7m
lab-getting-started-k14s-w01-s001   L.. simple-app-9b466965b-jzpzs  Pod         cluster  4/4 t   ok  -   7m

Rs: Reconcile state
Ri: Reconcile information

5 resources

Succeeded
```

Note that it even knows about resources it did not directly create (such as ReplicaSet and Endpoints).

```execute-1
kapp logs -f -a simple-app
```

```
# starting tailing 'simple-app-6f884d8d9d-nn5ds > simple-app' logs
simple-app-6f884d8d9d-nn5ds > simple-app | 2019/05/09 20:43:36 Server started
```

`inspect` and `logs` commands demonstrate why it's convenient to view resources in "bulk" (via a label). For example, `logs` command will tail any existing or new Pod that is part of simple-app application, even after we make changes and redeploy.

Let's stop that log tailing and continue with the workshop.

```execute-1
<ctrl-c>
```