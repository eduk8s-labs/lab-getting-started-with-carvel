Given that kapp tracks all resources that were deployed to k8s cluster, deleting them is as easy as running kapp delete command:

```execute-1
kapp delete -a simple-app
```

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

We can verify now our namespace is empty:

```execute-1
kubectl get all
```