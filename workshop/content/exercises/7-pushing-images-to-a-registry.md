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

The key parts of the output which of interest are:

```
...
quay.io/eduk8s-labs/sample-app-go | starting push (using Docker): kbld:docker-io-dkalinin-k8s-simple-app-sha256-268c33c1257eed727937fb22a68b91f065bf1e10c7ba23c5d897f2a2ab67f76d -> quay.io/eduk8s-labs/sample-app-go
quay.io/eduk8s-labs/sample-app-go | The push refers to repository [quay.io/eduk8s-labs/sample-app-go]
quay.io/eduk8s-labs/sample-app-go | 2c82b4929a5c: Preparing
quay.io/eduk8s-labs/sample-app-go | 2c82b4929a5c: Layer already exists
quay.io/eduk8s-labs/sample-app-go | latest: digest: sha256:4c8b96d4fffdfae29258d94a22ae4ad1fe36139d47288b8960d9958d1e63a9d0 size: 528
quay.io/eduk8s-labs/sample-app-go | finished push (using Docker)
resolve | final: quay.io/eduk8s-labs/sample-app-go -> index.quay.io/eduk8s-labs/sample-app-go@sha256:4c8b96d4fffdfae29258d94a22ae4ad1fe36139d47288b8960d9958d1e63a9d0
--- update deployment/simple-app (apps/v1) namespace: default
  ...
 30, 30             value: carvel user
 31     -         image: kbld:docker-io-dkalinin-k8s-simple-app-sha256-f999be3e0d96c78dc4d4c8330c8de8aff3c91f5e152f021d01cb3cd0e92a1797
     31 +         image: index.docker.io/your-username/your-repo@sha256:4c8b96d4fffdfae29258d94a22ae4ad1fe36139d47288b8960d9958d1e63a9d0
 32, 32           name: simple-app
 33, 33   status:

Changes

Namespace  Name        Kind        Conditions  Age  Changed  Ignored Reason
default    simple-app  Deployment  2 OK / 2    1d   mod      -

0 add, 0 delete (13 hidden), 1 update, 0 keep (1 hidden)

1 changes
...
```

This time the build should be successful.

If we inspect again the application we see the new referenced image:

```execute-1
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

For example:

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

A benefit of using `kbld` you will see is that the image digest reference (e.g. {{ registry_host }}/carvel/sample-app-go@sha256:4c8b96...) was used instead of a tagged reference (e.g. kbld:docker-io...).

Digest references are preferred to other image reference forms as they are `immutable`, hence provide a guarantee that the exact version of built software will be deployed.
