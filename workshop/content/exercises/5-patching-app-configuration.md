In addition to templating, `ytt` offers another way to customize application configuration.

Instead of relying on the provider of an application using templating to expose a set of configuration knobs, configuration consumers can use the [ytt overlay](https://github.com/k14s/ytt/blob/master/docs/lang-ref-ytt-overlay.md) feature to apply arbitrary patches to existing YAML configuration resources.

For example, our simple application configuration templates do not provide a way to parameterize the value of `spec.replicas`, which controls how may instances (pods) of the application should be run.

Instead of asking an author of an application to expose yet another data value input for a template, we can create an overlay file that changes `spec.replicas` to a new value.

An example configuration for `ytt` which does this can be found in the `config-step-2a-overlays/custom-scale.yml` file.

To view the full file run:

```execute
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

```execute
ytt template -f config-step-2-template/ -f config-step-2a-overlays/custom-scale.yml -v hello_msg="carvel user" | kapp deploy -a simple-app -f- --diff-changes --yes
```

The output should be similar to:

```
@@ update deployment/simple-app (apps/v1) namespace: {{session_namespace}} @@
  ...
 15, 15   spec:
     16 +   replicas: 2
 16, 17     selector:
 17, 18       matchLabels:

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
{{session_namespace}}  simple-app  Deployment  2/2 t   9m   update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

7:12:41PM: ---- applying 1 changes [0/1 done] ----
7:12:41PM: update deployment/simple-app (apps/v1) namespace: {{session_namespace}}
7:12:41PM: ---- waiting on 1 changes [0/1 done] ----
7:12:43PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
7:12:43PM:  ^ Waiting for generation 6 to be observed
7:12:43PM:  L ok: waiting on replicaset/simple-app-df7bbcb86 (apps/v1) namespace: {{session_namespace}}
7:12:43PM:  L ok: waiting on replicaset/simple-app-7d5ddf7d5b (apps/v1) namespace: {{session_namespace}}
7:12:43PM:  L ok: waiting on pod/simple-app-7d5ddf7d5b-rp78f (v1) namespace: {{session_namespace}}
7:12:43PM:  L ongoing: waiting on pod/simple-app-7d5ddf7d5b-rlsck (v1) namespace: {{session_namespace}}
7:12:43PM:     ^ Pending: ContainerCreating
7:12:45PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
7:12:45PM:  ^ Waiting for 1 unavailable replicas
7:12:45PM:  L ok: waiting on replicaset/simple-app-df7bbcb86 (apps/v1) namespace: {{session_namespace}}
7:12:45PM:  L ok: waiting on replicaset/simple-app-7d5ddf7d5b (apps/v1) namespace: {{session_namespace}}
7:12:45PM:  L ok: waiting on pod/simple-app-7d5ddf7d5b-rp78f (v1) namespace: {{session_namespace}}
7:12:45PM:  L ongoing: waiting on pod/simple-app-7d5ddf7d5b-rlsck (v1) namespace: {{session_namespace}}
7:12:45PM:     ^ Pending: ContainerCreating
7:12:47PM: ok: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
7:12:47PM: ---- applying complete [1/1 done] ----
7:12:47PM: ---- waiting complete [1/1 done] ----

Succeeded
```

To verify that we now have 2 instances of our application, run:

```execute-2
kubectl get pods
```
