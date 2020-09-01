Given that `kapp` tracks all resources that were deployed to the Kubernetes cluster, deleting them is as easy as running the `kapp` delete command:

```execute
kapp delete -a simple-app
```

When prompted, confirm that you do indeed want to delete all the resources:

```terminal:input
text: y
```

When complete the output should be:

```
Changes

Namespace  Name        Kind        Conditions  Age  Changed  Ignored Reason
default    simple-app  Deployment  2 OK / 2    1d   del      -
~          simple-app  Service     -           1d   del      -

0 add, 2 delete (13 hidden), 0 update, 0 keep

2 changes

Continue? [yN]: y
...
```

To verify that the namespace is empty run:

```execute
kubectl get all
```
