The previous section showed how to use kbld with local cluster that's backed by local Docker daemon. No remote registry was involved; however, for a production environment or in absence of a local environment, you will need to instruct kbld to push out built images to a registry accessible to your cluster. This is our case and we will finally see our application change.

`config-step-4-build-local/push.yml` specifies that `quay.io/eduk8s-labs/sample-app-go` should be pushed to a repository as specified by `push_images_repo` data value.

Our local docker client is already authenticated to the registry we will be pushing to, but otherwise you would need to make sure that it can push to it.

Also, to prepare for the deployment there is a secret that we need to apply once, so that Kubernetes can pull images from the private repo we have been using:

```execute-1
kubectl create secret generic registry-credentials --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson
```

Now 

```execute-1
ytt template -f config-step-4-build-and-push/ -v hello_msg="carvel user" -v push_images=true -v push_images_repo={{ REGISTRY_HOST }}/carvel/sample-app-go | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes
```

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

If we inspect again the application we see the new referenced image:

```execute-1
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

```
Images

Image     {{ REGISTRY_HOST }}/carvel/sample-app-go@sha256:c293f506529ee65e1f6c3600398d29b7677c5fa80c065f60354486dee28cb51a
Metadata  - Path: /home/eduk8s/exercises/sample-app-go
            Type: local
          - Dirty: true
            RemoteURL: https://github.com/eduk8s-labs/sample-app-go
            SHA: b677913bc9e92c45d6136b776bce011b45666619
            Type: git
Resource  deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s005

1 images

Succeeded
```

As a benefit of using __kbld__, you will see that image digest reference (e.g. {{ REGISTRY_HOST }}/carvel/sample-app-go@sha256:4c8b96...) was used instead of a tagged reference (e.g. kbld:docker-io...).

Digest references are preferred to other image reference forms as they are `immutable`, hence provide a gurantee that exact version of built software will be deployed.
