Once deployed successfully, you could access the application at 127.0.0.1:8080 in your browser with the help of kubectl port-forward command:

```
kubectl port-forward svc/simple-app 8080:8080
```

One downside to the kubectl command above: it has to be restarted if the application pod is recreated.

Alternatively, you could use [Carvel's kwt tool](https://github.com/k14s/kwt) which exposes cluster IP subnets and cluster DNS to your machine. This way, you can access the application without requiring any restarts.

With kwt installed, run the following command

```
sudo -E kwt net start 
and open http://simple-app.{{ session_namespace }}.svc.cluster.local/.
```

But as this workshop environment is already running inside the Kubernetes cluster, you can just access the service URL directly:

```execute-2
curl http://simple-app.{{ session_namespace }}.svc.cluster.local:8080
```

Let's now stop that log tailing for our application and continue with the workshop.

```execute-1
<ctrl-c>
```