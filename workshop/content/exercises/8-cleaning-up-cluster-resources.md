Given that `kapp` tracks all resources that were deployed to the Kubernetes cluster, deleting them is as easy as running the `kapp` delete command:

```execute
kapp delete -a simple-app
```

The initial output should be similar to:

```
Changes

Namespace           Name                         Kind        Conds.  Age  Op      Op st.  Wait to  Rs  Ri  
{{session_namespace}}  simple-app                   Deployment  2/2 t   2m   delete  -       delete   ok  -  
^                   simple-app                   Endpoints   -       2m   -       -       delete   ok  -  
^                   simple-app                   Service     -       2m   delete  -       delete   ok  -  
^                   simple-app-55485d45d7        ReplicaSet  -       2m   -       -       delete   ok  -  
^                   simple-app-7876bd7c99        ReplicaSet  -       1m   -       -       delete   ok  -  
^                   simple-app-7876bd7c99-5dzw2  Pod         4/4 t   1m   -       -       delete   ok  -  

Op:      0 create, 2 delete, 0 update, 4 noop
Wait to: 0 reconcile, 6 delete, 0 noop

Continue? [yN]: 
```

When prompted, confirm that you do indeed want to delete all the resources:

```terminal:input
text: y
```

To verify that the namespace is empty run:

```execute
kubectl get all
```
