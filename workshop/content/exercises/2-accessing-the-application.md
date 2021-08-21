# Accessing an application

For this next series of command executions we will need to switch between multiple terminal sessions.

Please type

```
screen
```
> Press the [space] bar.  You will be brought back to the current terminal session prompt.


As with this deployment we have only created a service to expose the deployment internally to the cluster, and have not exposed it using an external load balancer or ingress, in order to access it from your local computer, you would need to set up port forwarding. This can be done using the `kubectl` command by running:

```
kubectl port-forward svc/simple-app 8080:8080
```

You should see something like

```
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

The application is now exposed locally at `http://localhost:8080`.

To create a new terminal session, please press `Ctrl+a`, then `Ctrl+c`.

Then issue a request by running:

```
curl http://localhost:8080 && echo
```

and you'll see a response like

```
<h1>Hello stranger!</h1>
```

One downside to using `kubectl port-forward` to set up port forwarding is that if the pod is killed, you need to stop the port forwarding and recreate the tunnel, it will not rebind automatically to any new pod created to replace the original pod the tunnel was connected to.

Kill the pod by running:

```
kubectl delete pods -l simple-app
```

and once it has stopped, try using `curl` again, it will not work.

```
curl http://localhost:8080
```

Carvel provides a tool called [kwt](https://github.com/vmware-tanzu/carvel-kwt) to improve on the experience when using port forwarding.

Imagine you were using Linux or macOS and were connected to a Kubernetes cluster. You could run `kwt` on your local computer using:

```
sudo -E kwt net start
```

What this will do is expose the internal Kubernetes cluster IP subnet and cluster DNS to your local machine. This will allow you to access any service in the Kubernetes cluster as if you were inside of the Kubernetes cluster.

Go ahead and try it! Then press `Ctrl+a` and `Ctrl+a` again to toggle terminal sessions.

We need to exit the original failed `kubectl port-forward` by pressing `Ctrl+c`.

Now we can try the curl again except his time we'll reference the application by its DNS service name:

```
curl http://simple-app.default.svc.cluster.local:8080 && echo
```

In fact, any service in the Kubernetes cluster could be accessed from your local computer using the `svc.cluster.local` subnet. Because the IP subnet is being exposed, you don't have to worry about restarting port forwarding when a pod is killed.
