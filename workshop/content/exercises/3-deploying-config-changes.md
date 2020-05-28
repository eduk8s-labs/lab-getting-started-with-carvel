Let's make a change to the application configuration to simulate a common occurrence in a development workflow. A simple observable change we can make is to change the value of the `HELLO_MSG` environment variable in `config-step-1-minimal/config.yml`

```
         - name: HELLO_MSG
-          value: stranger
+          value: somebody
```

Go ahead and make this change in the embedded editor or using `vi` as you prefer. Once you've changed this file re-run kapp deploy:

```execute-1
kapp deploy -a simple-app -f config-step-1-minimal/ --diff-changes
```

__NOTE__: Remember again to Continue with the changes. In the future, we will provide a `-y` or `--yes` flag to go ahead with the changes without prompting us.

You should see a similar output to these:

```
@@ update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001 @@
  ...
 29, 29           - name: HELLO_MSG
 30     -           value: stranger
     30 +           value: somebody
 31, 31           image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
 32, 32           name: simple-app

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
lab-getting-started-k14s-w01-s001  simple-app  Deployment  2/2 t   6m   update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

Continue? [yN]: y

6:18:10PM: ---- applying 1 changes [0/1 done] ----
6:18:10PM: update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:18:11PM: ---- waiting on 1 changes [0/1 done] ----
6:18:12PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:18:12PM:  ^ Waiting for generation 4 to be observed
6:18:12PM:  L ok: waiting on replicaset/simple-app-6f78fb84cd (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:18:12PM:  L ok: waiting on replicaset/simple-app-5f4ddc8f6d (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:18:12PM:  L ok: waiting on pod/simple-app-6f78fb84cd-zsmr9 (v1) namespace: lab-getting-started-k14s-w01-s001
6:18:12PM:  L ongoing: waiting on pod/simple-app-5f4ddc8f6d-vqlsp (v1) namespace: lab-getting-started-k14s-w01-s001
6:18:12PM:     ^ Pending: ContainerCreating
6:18:14PM: ok: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:18:14PM: ---- applying complete [1/1 done] ----
6:18:14PM: ---- waiting complete [1/1 done] ----

Succeeded
```

Above output highlights several kapp features:

* kapp detected a single change to `simple-app` Deployment by comparing given local configuration against the live cluster copy
* kapp showed changes in a git-style diff via --diff-changes flag
* since `simple-app` Service was not changed in any way, it was not "touched" during the apply changes phase at all
* kapp waited for Pods associated with a Deployment to converge to their ready state before exiting successfully

To double check that our change applied, go ahead and hit the service url again in the terminal.

```execute-2
curl http://simple-app.{{ session_namespace }}.svc.cluster.local:8080
```

Given that kapp does not care where application configuration comes from, one can use it with any other tools that produce k8s configuration, for example, Helm's template command:

```
helm template my-chart --values values.yml | kapp deploy -a my-app -f- --yes
```