ytt also offers another way to customize application configuration. Instead of relying on configuration providers (e.g. authors of k8s-simple-app) to expose a set of configuration knobs, configuration consumers (e.g. users that deploy k8s-simple-app) can use the [ytt overlay](https://github.com/k14s/ytt/blob/master/docs/lang-ref-ytt-overlay.md) feature to patch YAML documents with arbitrary changes.

For example, our simple app configuration templates do not make Deployment's `spec.replicas` configurable as a data value to control how may Pods are running. Instead of asking authors of simple app to expose a new data value, we can create an overlay file `config-step-2a-overlays/custom-scale.yml` that changes `spec.replicas` to a new value.

```execute-1
ytt template -f config-step-2-template/ -f config-step-2a-overlays/custom-scale.yml -v hello_msg="carvel user" | kapp deploy -a simple-app -f- --diff-changes --yes
```

```
@@ update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001 @@
  ...
 15, 15   spec:
     16 +   replicas: 2
 16, 17     selector:
 17, 18       matchLabels:

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
lab-getting-started-k14s-w01-s001  simple-app  Deployment  2/2 t   9m   update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

7:12:41PM: ---- applying 1 changes [0/1 done] ----
7:12:41PM: update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:41PM: ---- waiting on 1 changes [0/1 done] ----
7:12:43PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:43PM:  ^ Waiting for generation 6 to be observed
7:12:43PM:  L ok: waiting on replicaset/simple-app-df7bbcb86 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:43PM:  L ok: waiting on replicaset/simple-app-7d5ddf7d5b (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:43PM:  L ok: waiting on pod/simple-app-7d5ddf7d5b-rp78f (v1) namespace: lab-getting-started-k14s-w01-s001
7:12:43PM:  L ongoing: waiting on pod/simple-app-7d5ddf7d5b-rlsck (v1) namespace: lab-getting-started-k14s-w01-s001
7:12:43PM:     ^ Pending: ContainerCreating
7:12:45PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:45PM:  ^ Waiting for 1 unavailable replicas
7:12:45PM:  L ok: waiting on replicaset/simple-app-df7bbcb86 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:45PM:  L ok: waiting on replicaset/simple-app-7d5ddf7d5b (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:45PM:  L ok: waiting on pod/simple-app-7d5ddf7d5b-rp78f (v1) namespace: lab-getting-started-k14s-w01-s001
7:12:45PM:  L ongoing: waiting on pod/simple-app-7d5ddf7d5b-rlsck (v1) namespace: lab-getting-started-k14s-w01-s001
7:12:45PM:     ^ Pending: ContainerCreating
7:12:47PM: ok: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
7:12:47PM: ---- applying complete [1/1 done] ----
7:12:47PM: ---- waiting complete [1/1 done] ----

Succeeded
```

We can verify that now we have 2 instances running of our application:

```execute-2
kubectl get pods
```