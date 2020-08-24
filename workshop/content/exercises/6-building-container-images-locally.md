K8s embraced use of container images to package source code and its dependencies. One way to deliver updated application is to rebuild a container when changing source code. [kbld](https://get-kbld.io/) is a small tool that provides a simple way to insert container image building into deployment workflow. kbld looks for images within application configuration (currently it looks for image keys), checks if there is an associated source code, if so builds these images via Docker (could be pluggable with other builders), and finally captures built image digests and updates configuration with new references.

As we are going to build our image locally, we will pull down the source code locally:

```execute-1
git clone https://github.com/eduk8s-labs/sample-app-go
```

This has created a new folder called `sample-app-go` where we can find the source code for the application.

Before running kbld, let's change `app.go` by adding a line to make a small change in our application. Add the following snippet after `line 14` in that file.

```
fmt.Fprintf(w, "<p>local change</p>\n")
```

`config-step-3-build-local/build.yml` is a new file in this config directory, which specifies that `quay.io/eduk8s-labs/sample-app-go` should be built from the current working directory where kbld runs (root of the repo).

__NOTE__: As we're using a real cluster, just read on but you may have to wait until the next section when we show how to use a remote registry.

Let's insert kbld between ytt and kapp so that images used in our configuration are built before they are deployed by kapp:

```execute-1
ytt template -f config-step-3-build-local/ -v hello_msg="carvel user" | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes
```

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

@@ update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001 @@
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
lab-getting-started-k14s-w01-s001  simple-app  Deployment  2/2 t   6m   update  reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

11:41:16AM: ---- applying 1 changes [0/1 done] ----
11:41:16AM: update deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:16AM: ---- waiting on 1 changes [0/1 done] ----
11:41:18AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:  ^ Waiting for generation 10 to be observed
11:41:18AM:  L ok: waiting on replicaset/simple-app-8d7df8bb8 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:  L ok: waiting on replicaset/simple-app-7b95fdff84 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:  L ok: waiting on replicaset/simple-app-6cbfc8b6d6 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:  L ok: waiting on replicaset/simple-app-5bf54866b6 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:  L ok: waiting on pod/simple-app-7b95fdff84-4v952 (v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:  L ongoing: waiting on pod/simple-app-6cbfc8b6d6-q4d7w (v1) namespace: lab-getting-started-k14s-w01-s001
11:41:18AM:     ^ Pending: ContainerCreating
11:41:20AM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:  ^ Waiting for 1 unavailable replicas
11:41:20AM:  L ok: waiting on replicaset/simple-app-8d7df8bb8 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:  L ok: waiting on replicaset/simple-app-7b95fdff84 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:  L ok: waiting on replicaset/simple-app-6cbfc8b6d6 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:  L ok: waiting on replicaset/simple-app-5bf54866b6 (apps/v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:  L ok: waiting on pod/simple-app-7b95fdff84-4v952 (v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:  L ongoing: waiting on pod/simple-app-6cbfc8b6d6-q4d7w (v1) namespace: lab-getting-started-k14s-w01-s001
11:41:20AM:     ^ Pending: ImagePullBackOff (message: Back-off pulling image "kbld:quay-io-eduk8s-labs-sample-app-go-sha256-03eb944c407c455f8966623ed600f157c23633577d02e51
a6763ba1dd546e471")
...
```

__NOTE__: As we're using a real cluster, our deployment can't access the image we just built. This command will not finish, so you will need to exist manually.

```execute-1
<ctrl-c>
```

As you can see, the above output shows that kbld received ytt's produced configuration, and used the docker build command to build simple app image, ultimately capturing a specific reference and passing it onto kapp.

If the deploy was successful you could check out application in your browser or via curl to see the updated response, but this is not our case.

It's also worth showing that kbld not only builds images and updates references but also annotates Kubernetes resources with image metadata it collects and makes it quickly accessible for debugging. This may not be that useful during development but comes handy when investigating environment (staging, production, etc.) state.

```execute-1
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

```
Images

Image     kbld:quay-io-eduk8s-labs-sample-app-go-sha256-7aed69a761f772d74416df18f92db11f547f70a54c9ab02592dc339d1d6bede2
Metadata  - Path: /home/eduk8s/exercises/sample-app-go
            Type: local
          - Dirty: true
            RemoteURL: https://github.com/eduk8s-labs/sample-app-go
            SHA: b677913bc9e92c45d6136b776bce011b45666619
            Type: git
Resource  deployment/simple-app (apps/v1) namespace: lab-getting-started-k14s-w01-s003

1 images

Succeeded
```