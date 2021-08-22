# Cleaning up

Given that `kapp` tracks all resources that were deployed to the Kubernetes cluster, deleting them is as easy as running the `kapp` delete command:

```
kapp delete -a simple-app
```

The initial output should be similar to:

```
Changes

Namespace  Name                         Kind           Conds.  Age  Op      Op st.  Wait to  Rs  Ri
default    simple-app                   Deployment     2/2 t   16h  delete  -       delete   ok  -
^          simple-app                   Endpoints      -       16h  -       -       delete   ok  -
^          simple-app                   Service        -       16h  delete  -       delete   ok  -
^          simple-app-57d74b4774        ReplicaSet     -       15h  -       -       delete   ok  -
^          simple-app-58d58b6967        ReplicaSet     -       4m   -       -       delete   ok  -
^          simple-app-58d58b6967-qzhrk  Pod            4/4 t   4m   -       -       delete   ok  -
^          simple-app-677b96597b        ReplicaSet     -       16h  -       -       delete   ok  -
^          simple-app-688bf88d9b        ReplicaSet     -       15h  -       -       delete   ok  -
^          simple-app-77d4cfc7b         ReplicaSet     -       49m  -       -       delete   ok  -
^          simple-app-drqd7             EndpointSlice  -       16h  -       -       delete   ok  -

Op:      0 create, 2 delete, 0 update, 8 noop
Wait to: 0 reconcile, 10 delete, 0 noop

Continue? [yN]:
```

When prompted, confirm that you do indeed want to delete all the resources:

Type `y` now.

We'll also stop kwt with

```
sudo -E kwt net clean-up
```

To verify that the namespace is empty run:

```
kubectl get all
```
