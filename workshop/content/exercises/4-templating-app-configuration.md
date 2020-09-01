Managing application configuration is a hard problem. As an application matures, typically configuration needs to be tweaked for different environments, and different constraints. This leads to the desire to expose several, hopefully not too many, configuration knobs that could be tweaked at the time of the deployment.

This problem is typically solved in two ways: templating or patching. Tools which exist to solve this problem are `helm` and `kustomize`. The Carvel project also has its own tool called [ytt](https://get-ytt.io/), which supports both approaches.

In this section we will see how `ytt` allows you to templatize YAML configuration, and in the next section, we'll see how it can patch YAML configuration via overlays.

Unlike many [other tools used for templating](https://github.com/k14s/ytt/blob/master/docs/ytt-vs-x.md#ytt-vs-x), `ytt` takes a different approach to working with YAML files. Instead of interpreting YAML configuration as plain text, it works with YAML structures such as maps, lists, YAML documents, scalars, etc. By doing so `ytt` is able to eliminate a lot of problems that plague other tools (character escaping, ambiguity, etc.).

On top that, `ytt` provides a Python-like language ([Starlark](https://github.com/bazelbuild/starlark)) that executes in a hermetic environment, making it friendly, yet more deterministic compared to intermixing a general purpose language with a template directly, or other non-familiar custom templating languages.

Take a look at [ytt: The YAML Templating Tool that simplifies complex configuration management](https://developer.ibm.com/blogs/yaml-templating-tool-to-simplify-complex-configuration-management/) for a more detailed introduction.

To tie it all together, let's take a look at `config-step-2-template/config.yml`. You can view the file by running:

```execute
cat config-step-2-template/config.yml
```

You should immediately notice that YAML comments (#@ ...) are used to store templating metadata within the YAML file. For example:

```
        env:
        - name: HELLO_MSG
          value: #@ data.values.hello_msg
```

The above snippet tells `ytt` that the `HELLO_MSG` environment variable value should be set to the value of `data.values.hello_msg`.

The `data.values` structure comes from the builtin `data` library of `ytt`, that allows us to expose configuration knobs through a separate file, namely `config-step-2-template/values.yml`. To view the values file, run:

```execute
cat config-step-2-template/values.yml
```

Deployers of `simple-app` can now decide, for example, what hello message to set without making application code or configuration changes.

Let's chain ytt and kapp to deploy an update, and note `-v` flag which sets hello_msg value:

```execute-1
ytt template -f config-step-2-template/ -v hello_msg="carvel user" | kapp deploy -a simple-app -f- --diff-changes --yes
```

```
@@ update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001 @@
  ...
 29, 29           - name: HELLO_MSG
 30     -           value: somebody
     30 +           value: carvel user
 31, 31           image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
 32, 32           name: simple-app

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
lab-getting-started-k14s-w01-s001  simple-app  Deployment  2/2 t   19m  update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

6:31:32PM: ---- applying 1 changes [0/1 done] ----
6:31:32PM: update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:31:32PM: ---- waiting on 1 changes [0/1 done] ----
6:31:34PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:31:34PM:  ^ Waiting for generation 10 to be observed
6:31:34PM:  L ok: waiting on replicaset/simple-app-c68644f6 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:31:34PM:  L ok: waiting on replicaset/simple-app-6f78fb84cd (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:31:34PM:  L ok: waiting on replicaset/simple-app-5f4ddc8f6d (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:31:34PM:  L ongoing: waiting on pod/simple-app-c68644f6-pf7mb (v1) namespace: lab-getting-started-k14s-w01-s001
6:31:34PM:     ^ Pending
6:31:34PM:  L ok: waiting on pod/simple-app-5f4ddc8f6d-7xx8r (v1) namespace: lab-getting-started-k14s-w01-s001
6:31:36PM: ok: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
6:31:36PM: ---- applying complete [1/1 done] ----
6:31:36PM: ---- waiting complete [1/1 done] ----

Succeeded
```

We can verify again the change:

```execute-2
curl http://simple-app.{{ session_namespace }}.svc.cluster.local:8080
```

We covered one simple way to use ytt to help you manage application configuration. Please take a look at examples in [ytt interactive playground](https://get-ytt.io/#playground) to learn more about other ytt features which may help you manage YAML configuration more effectively.
