Before getting too deep, let's get some basic preparations out of the way:

* __Find a Kubernetes cluster__. This workshop is already deployed in a Kubernetes cluster ready for you to use. If you want to replicate this workshop locally you can use Minikube or Docker for Mac/Windows.
* __Check that the cluster works with kubectl__

```execute
kubectl version
```

__NOTE__: Did you type the command in yourself? If you did, click on the command instead and you will find that it is executed for you. You can click on any command which has the <span class="fas fa-running"></span> icon shown to the right of it, and it will be copied to the interactive terminal and run. If you would rather make a copy of the command so you can paste it to another window, hold down the shift key when you click on the command.

* __Install Carvel tools by following instructions on [https://carvel.dev/](https://carvel.dev/)__ In this environment we have already installed the tools and they are available in the PATH.

```execute
ytt --version
```

```execute
kbld --version
```

```execute
kapp --version
```
