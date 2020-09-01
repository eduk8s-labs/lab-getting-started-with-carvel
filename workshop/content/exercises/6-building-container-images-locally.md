Kubernetes embraced the use of container images to package source code and its dependencies. Container images can be created using tools such as `docker`, `pack` if using Buildpacks, or the Maven `spring-boot:build-image` command if developing Spring Boot applications.

To rebuild a container image when changing application source would typically involve manually running a build using one of these tools, and then pushing the resulting image to an image registry accessible to the Kubernetes cluster so it can be deployed. Any deployment resource would then need to be modified to use the specific version for the new image.

The [kbld](https://get-kbld.io/) tool from Carvel is a small tool that provides a simple way to insert container image building into a deployment workflow. `kbld` looks for images within application configuration (currently it looks for image keys), checks if there is an associated source code definition, and if so triggers a build of the container image using `docker` (or other defined build mechanism), then finally captures the image digests for any built images, and updates configuration with new image references.

As we are going to build our image locally, we first need to pull down the source code:

```execute
git clone https://github.com/eduk8s-labs/sample-app-go
```

This has created a new folder called `sample-app-go` where you can find the source code for the application.

The configuration file `config-step-3-build-local/build.yml` is a new configuration, which specifies that the image `quay.io/eduk8s-labs/sample-app-go` should be built from source code in the `sample-app-go` sub directory when `kbld` runs. To view the configuration file run:

```execute
cat config-step-3-build-local/build.yml
```

You should see:

```
apiVersion: kbld.k14s.io/v1alpha1
kind: Sources
sources:
- image: quay.io/eduk8s-labs/sample-app-go
  path: sample-app-go
```

Now insert `kbld` between `ytt` and `kapp` so that images used in our configuration are built, if necessary, before they are deployed by `kapp`.

```execute-1
ytt template -f config-step-3-build-local/ -v hello_msg="carvel user" | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes
```

The output should be similar to the following:

```
quay.io/eduk8s-labs/sample-app-go | starting build (using Docker): sample-app-go -> kbld:rand-1590666070030466586-8975333420-quay-io-eduk8s-labs-sample-app-go
quay.io/eduk8s-labs/sample-app-go | Sending build context to Docker daemon  81.41kB
quay.io/eduk8s-labs/sample-app-go | Step 1/9 : FROM golang:1.12 as builder
quay.io/eduk8s-labs/sample-app-go |  ---> ffcaee6f7d8b
quay.io/eduk8s-labs/sample-app-go | Step 2/9 : WORKDIR /src
quay.io/eduk8s-labs/sample-app-go |  ---> Using cache
quay.io/eduk8s-labs/sample-app-go |  ---> 6347878f73ba
quay.io/eduk8s-labs/sample-app-go | Step 3/9 : COPY . .
quay.io/eduk8s-labs/sample-app-go |  ---> 7896a587f75e
quay.io/eduk8s-labs/sample-app-go | Step 4/9 : RUN CGO_ENABLED=0 GOOS=linux go build -v -o app
quay.io/eduk8s-labs/sample-app-go |  ---> Running in 94bbefa0a305
quay.io/eduk8s-labs/sample-app-go | net
quay.io/eduk8s-labs/sample-app-go | internal/x/net/http/httpproxy
quay.io/eduk8s-labs/sample-app-go | net/textproto
quay.io/eduk8s-labs/sample-app-go | crypto/x509
quay.io/eduk8s-labs/sample-app-go | internal/x/net/http/httpguts
quay.io/eduk8s-labs/sample-app-go | crypto/tls
quay.io/eduk8s-labs/sample-app-go | net/http/httptrace
quay.io/eduk8s-labs/sample-app-go | net/http
quay.io/eduk8s-labs/sample-app-go | _/src
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container 94bbefa0a305
quay.io/eduk8s-labs/sample-app-go |  ---> d46b89a51fab
quay.io/eduk8s-labs/sample-app-go | Step 5/9 : FROM scratch as runtime
quay.io/eduk8s-labs/sample-app-go |  --->
quay.io/eduk8s-labs/sample-app-go | Step 6/9 : COPY --from=builder /src/app .
quay.io/eduk8s-labs/sample-app-go |  ---> 85316d4960f2
quay.io/eduk8s-labs/sample-app-go | Step 7/9 : EXPOSE 8080
quay.io/eduk8s-labs/sample-app-go |  ---> Running in 924d7d31f91d
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container 924d7d31f91d
quay.io/eduk8s-labs/sample-app-go |  ---> ed2a9264a4d2
quay.io/eduk8s-labs/sample-app-go | Step 8/9 : USER 1000
quay.io/eduk8s-labs/sample-app-go |  ---> Running in e0effc36bc38
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container e0effc36bc38
quay.io/eduk8s-labs/sample-app-go |  ---> 361d4bc394a7
quay.io/eduk8s-labs/sample-app-go | Step 9/9 : ENTRYPOINT ["/app"]
quay.io/eduk8s-labs/sample-app-go |  ---> Running in faed2f404d15
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container faed2f404d15
quay.io/eduk8s-labs/sample-app-go |  ---> 03eb944c407c
quay.io/eduk8s-labs/sample-app-go | Successfully built 03eb944c407c
quay.io/eduk8s-labs/sample-app-go | Successfully tagged kbld:rand-1590666070030466586-8975333420-quay-io-eduk8s-labs-sample-app-go
quay.io/eduk8s-labs/sample-app-go | Untagged: kbld:rand-1590666070030466586-8975333420-quay-io-eduk8s-labs-sample-app-go
quay.io/eduk8s-labs/sample-app-go | finished build (using Docker)
resolve | final: quay.io/eduk8s-labs/sample-app-go -> kbld:quay-io-eduk8s-labs-sample-app-go-sha256-03eb944c407c455f8966623ed600f157c23633577d02e51a6763ba1dd546e471

@@ update deployment/simple-app (apps/v1) namespace: {{session_namespace}} @@
  ...
  4,  4       deployment.kubernetes.io/revision: "3"
      5 +     kbld.k14s.io/images: |
      6 +       - Metas:
      7 +         - Path: /home/eduk8s/exercises/sample-app-go
      8 +           Type: local
      9 +         - Dirty: true
     10 +           RemoteURL: https://github.com/eduk8s-labs/sample-app-go
     11 +           SHA: b677913bc9e92c45d6136b776bce011b45666619
     12 +           Type: git
     13 +         URL: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-03eb944c407c455f8966623ed600f157c23633577d02e51a6763ba1dd546e471
  5, 14     creationTimestamp: "2020-05-28T11:34:56Z"
  6, 15     generation: 8
  ...
 15, 24   spec:
 16     -   replicas: 2
 17, 25     selector:
 18, 26       matchLabels:
  ...
 31, 39             value: carvel user
 32     -         image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
     40 +         image: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-03eb944c407c455f8966623ed600f157c23633577d02e51a6763ba1dd546e471
 33, 41           name: simple-app
 34, 42   status:

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
{{session_namespace}}  simple-app  Deployment  2/2 t   6m   update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

11:41:16AM: ---- applying 1 changes [0/1 done] ----
11:41:16AM: update deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:41:16AM: ---- waiting on 1 changes [0/1 done] ----
11:41:18AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:41:18AM:  ^ Waiting for generation 10 to be observed
11:41:18AM:  L ok: waiting on replicaset/simple-app-8d7df8bb8 (apps/v1) namespace: {{session_namespace}}
11:41:18AM:  L ok: waiting on replicaset/simple-app-7b95fdff84 (apps/v1) namespace: {{session_namespace}}
11:41:18AM:  L ok: waiting on replicaset/simple-app-6cbfc8b6d6 (apps/v1) namespace: {{session_namespace}}
11:41:18AM:  L ok: waiting on replicaset/simple-app-5bf54866b6 (apps/v1) namespace: {{session_namespace}}
11:41:18AM:  L ok: waiting on pod/simple-app-7b95fdff84-4v952 (v1) namespace: {{session_namespace}}
11:41:18AM:  L ongoing: waiting on pod/simple-app-6cbfc8b6d6-q4d7w (v1) namespace: {{session_namespace}}
11:41:18AM:     ^ Pending: ContainerCreating
11:41:20AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: {{session_namespace}}
11:41:20AM:  ^ Waiting for 1 unavailable replicas
11:41:20AM:  L ok: waiting on replicaset/simple-app-8d7df8bb8 (apps/v1) namespace: {{session_namespace}}
11:41:20AM:  L ok: waiting on replicaset/simple-app-7b95fdff84 (apps/v1) namespace: {{session_namespace}}
11:41:20AM:  L ok: waiting on replicaset/simple-app-6cbfc8b6d6 (apps/v1) namespace: {{session_namespace}}
11:41:20AM:  L ok: waiting on replicaset/simple-app-5bf54866b6 (apps/v1) namespace: {{session_namespace}}
11:41:20AM:  L ok: waiting on pod/simple-app-7b95fdff84-4v952 (v1) namespace: {{session_namespace}}
11:41:20AM:  L ongoing: waiting on pod/simple-app-6cbfc8b6d6-q4d7w (v1) namespace: {{session_namespace}}
11:41:20AM:     ^ Pending: ImagePullBackOff (message: Back-off pulling image "kbld:quay-io-eduk8s-labs-sample-app-go-sha256-03eb944c407c455f8966623ed600f157c23633577d02e51
a6763ba1dd546e471")
...
```

Note that you will see that the deployment is failing. This is because we haven't configured `kbld` to push the image to an image registry where Kubernetes can pull it from. We will show the required steps to push the image to an image registry in the next section. For now, stop the attempted deployment by interrupting `kapp`.

```terminal:interrupt
```

What you can see from the above output though, is how `kbld` triggered a build of the container image using `docker` based on the requirement for the image in the configuration received from `ytt`. Further, `kbld` modified the image reference to include the new image digest before passing the configuration onto `kapp` for deployment.

It's also worth noting that `kbld` not only builds images and updates references but also annotates Kubernetes resources with image metadata it collects and makes it quickly accessible for debugging. This may not be that useful during development but comes handy when investigating environment (staging, production, etc.) state.

```execute-1
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

The output from this will be similar to:

```
Images

Image     kbld:quay-io-eduk8s-labs-sample-app-go-sha256-7aed69a761f772d74416df18f92db11f547f70a54c9ab02592dc339d1d6bede2
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
