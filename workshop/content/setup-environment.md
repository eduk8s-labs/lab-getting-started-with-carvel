# Environment Setup

This environment will provide you with a Kubernetes cluster and a private container image registry powered by Harbor to use when running the exercises covered by this workshop.

Let's invoke a startup script to bootstrap the cluster and install Harbor.

```
./startup.sh
```
> Plesae be patient.  This will take anywhere from 3-5 minutes to complete.

Let's verify that the cluster node and pods are up and running
```
kubectl get nodes
kubectl get pods -A
```

And quickly verify the versions of the CLIs we will leverage

```
kwt version
ytt version
kapp version
kbld version
imgpkg version
vendir version
```

If everything looks fine to you, then please continue to the first exercise.
