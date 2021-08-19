# Templating application configuration

Managing application configuration is a hard problem. As an application matures, typically configuration needs to be tweaked for different environments, and different constraints. This leads to the desire to expose several, hopefully not too many, configuration knobs that could be tweaked at the time of the deployment.

This problem is typically solved in two ways: templating or patching. Tools which exist to solve this problem are `helm` and `kustomize`. The Carvel project also has its own tool called [ytt](https://get-ytt.io/), which supports both approaches.

In this section we will see how `ytt` allows you to templatize YAML configuration, and in the next section, we'll see how it can patch YAML configuration via overlays.

Unlike many [other tools used for templating](https://carvel.dev/ytt/docs/latest/ytt-vs-x/), `ytt` takes a different approach to working with YAML files. Instead of interpreting YAML configuration as plain text, it works with YAML structures such as maps, lists, YAML documents, scalars, etc. By doing so `ytt` is able to eliminate a lot of problems that plague other tools (character escaping, ambiguity, etc.).

On top of that, `ytt` provides a Python-like language ([Starlark](https://github.com/bazelbuild/starlark)) that executes in a hermetic environment, making it friendly, yet more deterministic compared to intermixing a general purpose language with a template directly, or other non-familiar custom templating languages.

Take a look at [ytt: The YAML Templating Tool that simplifies complex configuration management](https://developer.ibm.com/blogs/yaml-templating-tool-to-simplify-complex-configuration-management/) for a more detailed introduction.

To tie all these statements together, let's take a look at `config-step-2-template/config.yml`. You can view the file by running:

```
cat config-step-2-template/config.yml
```

You should immediately notice that YAML comments (#@ ...) are used to store templating metadata within the YAML file. For example:

```
        env:
        - name: HELLO_MSG
          value: #@ data.values.hello_msg
```

The above snippet tells `ytt` that the `HELLO_MSG` environment variable value should be set to the value of `data.values.hello_msg`.

The `data.values` structure comes from the builtin `data` library of `ytt`, that allows us to expose configuration knobs through a separate file, namely `config-step-2-template/values.yml`.

To view the values file, run:

```
cat config-step-2-template/values.yml
```

The output should be:

```
#@data/values
---
svc_port: 8080
app_port: 8080
hello_msg: stranger
```

Someone deploying our `simple-app` application can now modify what `hello_msg` is set to without making changes to the application code or the main deployment resource, by editing it in the values file, or even by supplying an override.

Let's chain `ytt` and `kapp` to deploy an update to our application. Note how we can use the `-v` flag to `ytt` to override the data value for the hello message, thus avoiding need to change the values file.

```
ytt template -f config-step-2-template/ -v hello_msg="carvel user" | kapp deploy -a simple-app -f- --diff-changes --yes
```

The output should be as follows. Because we supplied `--yes` to `kapp`, the changes were immediately accepted and applied.

```
@@ update deployment/simple-app (apps/v1) namespace: default @@
  ...
117,117           - name: HELLO_MSG
118     -           value: somebody
    118 +           value: carvel user
119,119           image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
120,120           name: simple-app

Changes

Namespace  Name        Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
default    simple-app  Deployment  2/2 t   1h   update  -       reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

12:43:52AM: ---- applying 1 changes [0/1 done] ----
12:43:52AM: update deployment/simple-app (apps/v1) namespace: default
12:43:52AM: ---- waiting on 1 changes [0/1 done] ----
12:43:52AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
12:43:52AM:  ^ Waiting for generation 6 to be observed
12:43:52AM:  L ok: waiting on replicaset/simple-app-688bf88d9b (apps/v1) namespace: default
12:43:52AM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
12:43:52AM:  L ok: waiting on replicaset/simple-app-57d74b4774 (apps/v1) namespace: default
12:43:52AM:  L ongoing: waiting on pod/simple-app-688bf88d9b-nsjxw (v1) namespace: default
12:43:52AM:     ^ Pending
12:43:52AM:  L ok: waiting on pod/simple-app-57d74b4774-nfsk4 (v1) namespace: default
12:43:53AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
12:43:53AM:  ^ Waiting for 1 unavailable replicas
12:43:53AM:  L ok: waiting on replicaset/simple-app-688bf88d9b (apps/v1) namespace: default
12:43:53AM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
12:43:53AM:  L ok: waiting on replicaset/simple-app-57d74b4774 (apps/v1) namespace: default
12:43:53AM:  L ongoing: waiting on pod/simple-app-688bf88d9b-nsjxw (v1) namespace: default
12:43:53AM:     ^ Pending: ContainerCreating
12:43:53AM:  L ok: waiting on pod/simple-app-57d74b4774-nfsk4 (v1) namespace: default
12:43:54AM: ok: reconcile deployment/simple-app (apps/v1) namespace: default
12:43:54AM: ---- applying complete [1/1 done] ----
12:43:54AM: ---- waiting complete [1/1 done] ----

Succeeded
```

Verify the applications works, and that the response has changed by running:

```
curl http://simple-app.default.svc.cluster.local:8080 && echo
```

We covered one simple way to use `ytt` to help you manage application configuration. See the [ytt interactive playground](https://carvel.dev/ytt/#playground) to learn more about other `ytt` features which may help you manage YAML configuration more effectively.
