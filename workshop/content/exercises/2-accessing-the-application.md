As with this deployment we have only created a service to expose the deployment internally to the cluster, and have not exposed it using an external load balancer or ingress, in order to access it from your local computer, you would need to set up port forwarding. This can be done using the `kubectl` command by running:

```execute-1
kubectl port-forward svc/simple-app 8080:8080
```

The application will be exposed locally and accessible using `http://localhost:8080`. You can test this from the workshop environment by running:

```execute-2
curl http://localhost:8080
```

One downside to using `kubectl port-forward` to set up port fowarding is that if the pod is killed, you need to stop the port forwarding and recreate the tunnel, it will not rebind automatically to any new pod created to replace the original pod the tunnel was connected to.

Kill the pod by running:

```execute-2
kubectl delete pods -l simple-app
```

and once it has stopped, try using `curl` again, it will not work.

```execute-2
curl http://localhost:8080
```

Although we can't demonstrate it in this workshop environment, Carvel provides a tool called [kwt](https://github.com/k14s/kwt) to improve on the experience when using port forwarding.

If you were using Linux or macOS and were connected to a Kubernetes cluster, you could run `kwt` on your local computer using:

```
sudo -E kwt net start
```

What this will do is expose the internal Kubernetes cluster IP subnet and cluster DNS to your local machine. This will allow you to access any service in the Kubernetes cluster as if you were inside of the Kubernetes cluster.

For example, if using `kwt`, you would be able to access our application on your local machine using:

```
http://simple-app.{{session_namespace}}.svc.cluster.local:8080
```

That is, any service in the Kubernetes cluster could be accessed from your local computer using the `svc.cluster.local` subnet. Because the IP subnet is being exposed, you don't have to worry about restarting port forwarding when a pod is killed. 

Stop the current port forwarding as we no longer require it.

```terminal:interrupt
session: 1
```
