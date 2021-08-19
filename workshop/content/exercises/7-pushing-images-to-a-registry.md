# Pushing images to a registry

The previous section showed how to use `kbld` to build the container image. It failed to deploy in the end as the Kubernetes cluster being used didn't have access to images held by the docker daemon where the build was being run.

In order to have it work for the Kubernetes cluster used by this workshop environment, we will need to push the image to an image registry the cluster can access.

The configuration we will use this time is `config-step-4-build-and-push/push.yml`. To view the file run:

```
cat config-step-4-build-and-push/push.yml
```

The output should be:

```
#@ load("@ytt:data", "data")
#@ load("@ytt:assert", "assert")

#@ if/end data.values.push_images:
---
apiVersion: kbld.k14s.io/v1alpha1
kind: ImageDestinations
destinations:
- image: quay.io/eduk8s-labs/sample-app-go
  #@ if not data.values.push_images_repo or len(data.values.push_images_repo) == 0:
  #@   assert.fail("Expected push_images_repo to be non-empty. Example: quay.io/eduk8s-labs/sample-app-go")
  #@ end
  newImage: #@ data.values.push_images_repo
```

The configuration specifies that `quay.io/eduk8s-labs/sample-app-go` should be pushed to an image repository with name as specified by `push_images_repo` data value.

Our local docker client is already authenticated to the registry we will be pushing to, but otherwise you would need to make sure that it can push to it.

Also, to prepare for the deployment, we need to create an image pull secret to use with the deployment, so that Kubernetes can pull images from the private repo we are using:

```
kubectl create secret generic registry-credentials --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson
```

Run the combined command to process the template, build the image and deploy it, setting the value for `push_images_repo` in the process.

```
ytt template -f config-step-4-build-and-push/ -v hello_msg="carvel user" -v push_images=true -v push_images_repo=core.harbor.domain/library/sample-app-go | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes
```

The key parts of the output which are of interest are:

```
...
core.harbor.domain/library/sample-app-go | starting push (using Docker): kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5 -> core.harbor.domain/library/sample-app-go:kbld-rand-1629387768823219226-23522611324779
core.harbor.domain/library/sample-app-go | The push refers to repository [core.harbor.domain/library/sample-app-go]
core.harbor.domain/library/sample-app-go | a2994e8738e5: Preparing
core.harbor.domain/library/sample-app-go | a2994e8738e5: Pushed
core.harbor.domain/library/sample-app-go | kbld-rand-1629387768823219226-23522611324779: digest: sha256:5b2eda92ba2337c67beac54bec75dfe21ec3ad001f80230c647cabd5b8895655 size: 528
core.harbor.domain/library/sample-app-go | finished push (using Docker)
resolve | final: quay.io/eduk8s-labs/sample-app-go -> core.harbor.domain/library/sample-app-go@sha256:5b2eda92ba2337c67beac54bec75dfe21ec3ad001f80230c647cabd5b8895655

@@ update deployment/simple-app (apps/v1) namespace: default @@
  ...
 12, 12             Type: git
 13     -         URL: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5
     13 +         URL: core.harbor.domain/library/sample-app-go@sha256:5b2eda92ba2337c67beac54bec75dfe21ec3ad001f80230c647cabd5b8895655
 14, 14     creationTimestamp: "2021-08-18T23:29:52Z"
 15, 15     generation: 10
  ...
129,129             value: carvel user
130     -         image: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5
    130 +         image: core.harbor.domain/library/sample-app-go@sha256:5b2eda92ba2337c67beac54bec75dfe21ec3ad001f80230c647cabd5b8895655
131,131           name: simple-app
    132 +       imagePullSecrets:
    133 +       - name: registry-credentials
132,134   status:
133,135     availableReplicas: 1

Changes

Namespace  Name        Kind        Conds.  Age  Op      Op st.  Wait to    Rs    Ri
default    simple-app  Deployment  1/2 t   16h  update  -       reconcile  fail  Deployment is not progressing:
                                                                                 ProgressDeadlineExceeded (message:
                                                                                 ReplicaSet "simple-app-77d4cfc7b"
                                                                                 has timed out progressing.)

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

3:42:51PM: ---- applying 1 changes [0/1 done] ----
...
```

This time the build and deployment should be successful.

If we inspect again the application we see the new image being referenced:

```
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

The output should be similar to:

```
Images

Image     core.harbor.domain/library/sample-app-go@sha256:5b2eda92ba2337c67beac54bec75dfe21ec3ad001f80230c647cabd5b8895655
Metadata  - Path: /home/ubuntu/sample-app-go
            Type: local
          - Dirty: false
            RemoteURL: https://github.com/eduk8s-labs/sample-app-go
            SHA: 338d4d9e5299952a0bdb10d72b873ebbde0a1410
            Type: git
Resource  deployment/simple-app (apps/v1) namespace: default

1 images

Succeeded
```

You will note how the image name has been rewritten to reference the image from the image registry it was pushed to, rather than the original location for the image. Also you will see that the image digest reference (e.g. core.harbor.domain/library/sample-app-go@sha256:5b2eda...) was used instead of a tagged reference (e.g. kbld:docker-io...).

Digest references are preferred to other image reference forms as they are immutable, hence provide a guarantee that the exact version of built software will be deployed.
