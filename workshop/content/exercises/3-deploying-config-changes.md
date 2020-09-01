We mentioned previously that one of the benefits of `kapp` is that it can notify us of what changes are being made before we decide to accept the change. To show this in action, let's make a change to the application configuration to simulate a common occurrence in a development workflow.

If you remember, the deployment resource for our application in the file `config-step-1-minimal/config.yml` set an environment variable `HELLO_MSG`. This was set to "stranger". Change the value of this environment variable to "somebody" by running:

```execute
sed -ibak -e "s/stranger/somebody/" config-step-1-minimal/config.yml
```

Now re-run `kapp`, but this time add the `--diff-changes` option to tell it to show us what changes have been made.

```execute-1
kapp deploy -a simple-app -f config-step-1-minimal/ --diff-changes
```

You should see similar output to the following:

```
@@ update deployment/simple-app (apps/v1) namespace: {{session_namespace}} @@
  ...
 29, 29           - name: HELLO_MSG
 30     -           value: stranger
     30 +           value: somebody
 31, 31           image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
 32, 32           name: simple-app

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
{{session_namespace}}  simple-app  Deployment  2/2 t   6m   update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

Continue? [yN]: y
```

You can see how it displays the difference between what is currently being used and what you want it to be set to.

You will again be prompted as to whether you want to continue with the changes. Enter `y`.

```terminal:input
text: y
```

The change should then be applied.

```
6:18:10PM: ---- applying 1 changes [0/1 done] ----
6:18:10PM: update deployment/simple-app (apps/v1) namespace: {{session_namespace}}
6:18:11PM: ---- waiting on 1 changes [0/1 done] ----
6:18:12PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
6:18:12PM:  ^ Waiting for generation 4 to be observed
6:18:12PM:  L ok: waiting on replicaset/simple-app-6f78fb84cd (apps/v1) namespace: {{session_namespace}}
6:18:12PM:  L ok: waiting on replicaset/simple-app-5f4ddc8f6d (apps/v1) namespace: {{session_namespace}}
6:18:12PM:  L ok: waiting on pod/simple-app-6f78fb84cd-zsmr9 (v1) namespace: {{session_namespace}}
6:18:12PM:  L ongoing: waiting on pod/simple-app-5f4ddc8f6d-vqlsp (v1) namespace: {{session_namespace}}
6:18:12PM:     ^ Pending: ContainerCreating
6:18:14PM: ok: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
6:18:14PM: ---- applying complete [1/1 done] ----
6:18:14PM: ---- waiting complete [1/1 done] ----

Succeeded
```

In the future, we will provide a `-y` or `--yes` flag to go ahead with the changes without prompting us. The default though is to always prompt if you want to continue.

The above output highlights further features of `kapp`:

* `kapp` detects changes to resources by comparing the given local configuration against the live version already deployed to the cluster
* `kapp` can show changes in a git-style diff by supplying the `--diff-changes` flag
* `kapp` will only re-apply resources which were changed, so since the service resource was unchanged, it was not re-applied to the cluster during the apply changes phase
* `kapp` waited for pods associated with a deployment to converge to their ready state before exiting successfully

To double check that the change was applied, run:

```execute-2
curl http://simple-app.{{ session_namespace }}.svc.cluster.local:8080
```

> NOTE: This actually worked even though we hadn't set up port forwarding because this workshop environment is running in the same Kubernetes cluster as the deployment was made. You will still need to use `kubectl port-forward` or `kwt` on your local computer if that is where you were working from.

One of the key features of `kapp` mentioned above is that when detecting changes it will compare against the live version already deployed in the cluster. Because it compares against what is live in the Kubernetes cluster, `kapp` does not really care where the application configuration you want to apply comes from, you do not even need to have the new configuration stored as files in the filesystem.

What this means is that one can use `kapp` with any other tools that produce Kubernetes configuration, for example, if you were using `helm` you could run:

```
helm template my-chart --values values.yml | kapp deploy -a my-app -f- --yes
```

In this case the output from `helm` command is piped directly into `kapp`.

You can therefore start to see how the Carvel tools are designed in the UNIX philsophy of small individual tools designed to perform one task, that can be composed together to create more complex workflows. Such workflows are not restricted to being composed of just tools from the Carvel project.
