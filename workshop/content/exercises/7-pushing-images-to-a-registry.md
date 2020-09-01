The previous section showed how to use `kbld` to build the container image. It failed to deploy in the end as the Kubernetes cluster being used didn't have access to images held by docker daemon where the build was being run.

In order to have it work for the Kubernetes cluster used by this workshop environment, we will need to push the image to an image registry the cluster can access.

The configuration we will use this time is `config-step-4-build-and-push/push.yml`. To view the file run:

```execute
cat config-step-4-build-and-push/push.yml
```

The configuration specifies that `quay.io/eduk8s-labs/sample-app-go` should be pushed to an image repository with name as specified by `push_images_repo` data value.

Our local docker client is already authenticated to the registry we will be pushing to, but otherwise you would need to make sure that it can push to it.

Also, to prepare for the deployment, we need to create an image pull secret to use with the deployment, so that Kubernetes can pull images from the private repo we are using:

```execute
kubectl create secret generic registry-credentials --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson
```

Run the combined command to process the template, build the image and deploy it, setting the value for `push_images_repo` in the process.

```execute-1
ytt template -f config-step-4-build-and-push/ -v hello_msg="carvel user" -v push_images=true -v push_images_repo={{ registry_host }}/carvel/sample-app-go | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes
```

The key parts of the output which are of interest are:

```
...
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | starting push (using Docker): kbld:quay-io-eduk8s-labs-sample-app-go-sha256-8ce490851d3f58808a6fbe418dc3775719c1a5b694fef5529b1fc07d7d97cf57 -> {{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go:kbld-rand-1598962590309459451-219513548161
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | The push refers to repository [{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go]
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | aab6520b305d: Preparing
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | aab6520b305d: 
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | Pushed
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | kbld-rand-1598962590309459451-219513548161: digest: sha256:c9b355704f63e231a282309ffa6f314d56e1060453e123e50a462a4ff3fb0819 size: 528
{{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go | finished push (using Docker)
resolve | final: quay.io/eduk8s-labs/sample-app-go -> {{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go@sha256:c9b355704f63e231a282309ffa6f314d56e1060453e123e50a462a4ff3fb0819

@@ update deployment/simple-app (apps/v1) namespace: {{session_namespace}} @@
  ...
 12, 12             Type: git
 13     -         URL: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-8ce490851d3f58808a6fbe418dc3775719c1a5b694fef5529b1fc07d7d97cf57
     13 +         URL: {{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go@sha256:c9b355704f63e231a282309ffa6f314d56e1060453e123e50a462a4ff3fb0819
 14, 14     creationTimestamp: "2020-09-01T12:14:52Z"
 15, 15     generation: 2
  ...
132,132             value: carvel user
133     -         image: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-8ce490851d3f58808a6fbe418dc3775719c1a5b694fef5529b1fc07d7d97cf57
    133 +         image: {{session_namespace}}-registry.training.getwarped.org/carvel/sample-app-go@sha256:c9b355704f63e231a282309ffa6f314d56e1060453e123e50a462a4ff3fb0819
134,134           name: simple-app
    135 +       imagePullSecrets:
    136 +       - name: registry-credentials
135,137   status:
136,138     conditions:

Changes

Namespace                                 Name        Kind        Conds.  Age  Op      Op st.  Wait to    Rs       Ri  
{{session_namespace}}  simple-app  Deployment  1/2 t   1m   update  -       reconcile  ongoing  Waiting for 1 unavailable replicas  

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

12:16:33PM: ---- applying 1 changes [0/1 done] ----
...
```

This time the build should be successful.

If we inspect again the application we see the new referenced image:

```execute-1
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

The output should be similar to:

```
Images

Image     {{ registry_host }}/carvel/sample-app-go@sha256:c293f506529ee65e1f6c3600398d29b7677c5fa80c065f60354486dee28cb51a
Metadata  - Path: /home/eduk8s/exercises/sample-app-go
            Type: local
          - Dirty: true
            RemoteURL: https://github.com/eduk8s-labs/sample-app-go
            SHA: b677913bc9e92c45d6136b776bce011b45666619
            Type: git
Resource  deployment/simple-app (apps/v1) namespace: {{session_namespace}}

1 images

Succeeded
```

You will note how the image name has been rewritten to reference the image from the image registry it was pushed to, rather than the original location for the image. Also you will see that the image digest reference (e.g. {{ registry_host }}/carvel/sample-app-go@sha256:4c8b96...) was used instead of a tagged reference (e.g. kbld:docker-io...).

Digest references are preferred to other image reference forms as they are `immutable`, hence provide a guarantee that the exact version of built software will be deployed.
