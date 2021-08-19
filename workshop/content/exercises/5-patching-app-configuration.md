# Patching application configuration

In addition to templating, `ytt` offers another way to customize application configuration.

Instead of relying on the provider of an application using templating to expose a set of configuration knobs, configuration consumers can use the [ytt overlay](https://github.com/k14s/ytt/blob/master/docs/lang-ref-ytt-overlay.md) feature to apply arbitrary patches to existing YAML configuration resources.

For example, our simple application configuration templates do not provide a way to parameterize the value of `spec.replicas`, which controls how may instances (pods) of the application should be run.

Instead of asking an author of an application to expose yet another data value input for a template, we can create an overlay file that changes `spec.replicas` to a new value.

An example configuration for `ytt` which does this can be found in the `config-step-2a-overlays/custom-scale.yml` file.

To view the full file run:

```
cat config-step-2a-overlays/custom-scale.yml
```

The key part of the overlay file is:

```
#@overlay/match by=overlay.subset({"kind": "Deployment"})
---
spec:
  #@overlay/match missing_ok=True
  replicas: 2
```

This says that for any `Deployment` resource, patch `spec.replicas` so the resulting value is `2`. This will be added even if `spec.replicas` was not present in the first place.

To update the application deployment using this configuration run:

```
ytt template -f config-step-2-template/ -f config-step-2a-overlays/custom-scale.yml -v hello_msg="carvel user" | kapp deploy -a simple-app -f- --diff-changes --yes
```

The output should be similar to:

```
@@ update deployment/simple-app (apps/v1) namespace: default @@
  ...
103,103   spec:
    104 +   replicas: 2
104,105     selector:
105,106       matchLabels:

Changes

Namespace  Name        Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
default    simple-app  Deployment  2/2 t   1h   update  -       reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

12:46:53AM: ---- applying 1 changes [0/1 done] ----
12:46:53AM: update deployment/simple-app (apps/v1) namespace: default
12:46:53AM: ---- waiting on 1 changes [0/1 done] ----
12:46:53AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
12:46:53AM:  ^ Waiting for generation 8 to be observed
12:46:53AM:  L ok: waiting on replicaset/simple-app-688bf88d9b (apps/v1) namespace: default
12:46:53AM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
12:46:53AM:  L ok: waiting on replicaset/simple-app-57d74b4774 (apps/v1) namespace: default
12:46:53AM:  L ok: waiting on pod/simple-app-688bf88d9b-nsjxw (v1) namespace: default
12:46:53AM:  L ongoing: waiting on pod/simple-app-688bf88d9b-nlfnr (v1) namespace: default
12:46:53AM:     ^ Pending: ContainerCreating
12:46:54AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
12:46:54AM:  ^ Waiting for 1 unavailable replicas
12:46:54AM:  L ok: waiting on replicaset/simple-app-688bf88d9b (apps/v1) namespace: default
12:46:54AM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
12:46:54AM:  L ok: waiting on replicaset/simple-app-57d74b4774 (apps/v1) namespace: default
12:46:54AM:  L ok: waiting on pod/simple-app-688bf88d9b-nsjxw (v1) namespace: default
12:46:54AM:  L ongoing: waiting on pod/simple-app-688bf88d9b-nlfnr (v1) namespace: default
12:46:54AM:     ^ Pending: ContainerCreating
12:46:55AM: ok: reconcile deployment/simple-app (apps/v1) namespace: default
12:46:55AM: ---- applying complete [1/1 done] ----
12:46:55AM: ---- waiting complete [1/1 done] ----

Succeeded
```

To verify that we now have 2 instances of our application, run:

```
kubectl get pods -l simple-app
```

which should yield

```
NAME                          READY   STATUS    RESTARTS   AGE
simple-app-688bf88d9b-nlfnr   1/1     Running   0          6h54m
simple-app-688bf88d9b-nsjxw   1/1     Running   0          6h57m
```
