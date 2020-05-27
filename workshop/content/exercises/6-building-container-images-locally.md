K8s embraced use of container images to package source code and its dependencies. One way to deliver updated application is to rebuild a container when changing source code. [kbld](https://get-kbld.io/) is a small tool that provides a simple way to insert container image building into deployment workflow. kbld looks for images within application configuration (currently it looks for image keys), checks if there is an associated source code, if so builds these images via Docker (could be pluggable with other builders), and finally captures built image digests and updates configuration with new references.

As we are going to build our image locally, we will pull down the source code locally:

```
git clone https://github.com/eduk8s-labs/sample-app-go
```

This has created a new folder called `sample-app-go` where we can find the source code for the application.

Before running kbld, let's change `app.go` by adding a line to make a small change in our application. Add the following snippet after `line 14` in that file.

```
fmt.Fprintf(w, "<p>local change</p>"
```

`config-step-3-build-local/build.yml` is a new file in this config directory, which specifies that `quay.io/eduk8s-labs/sample-app-go` should be built from the current working directory where kbld runs (root of the repo).

__NOTE__: As we're using a real cluster, just read on but you may have to wait until the next section when we show how to use a remote registry.

Let's insert kbld between ytt and kapp so that images used in our configuration are built before they are deployed by kapp:

```
ytt template -f config-step-3-build-local/ -v hello_msg="k14s user" | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes

docker.io/dkalinin/k8s-simple-app | starting build (using Docker): . -> kbld:1557534242219453000-quay-io-eduk8s-labs-sample-app-go
docker.io/dkalinin/k8s-simple-app | Sending build context to Docker daemon  223.7kB
docker.io/dkalinin/k8s-simple-app | Step 1/8 : FROM golang:1.12 as builder
docker.io/dkalinin/k8s-simple-app |  ---> 7ced090ee82e
docker.io/dkalinin/k8s-simple-app | Step 2/8 : WORKDIR /src/
docker.io/dkalinin/k8s-simple-app |  ---> Using cache
docker.io/dkalinin/k8s-simple-app |  ---> cfcc02b178b8
docker.io/dkalinin/k8s-simple-app | Step 3/8 : COPY . .
docker.io/dkalinin/k8s-simple-app |  ---> 7c5f468a7d66
docker.io/dkalinin/k8s-simple-app | Step 4/8 : RUN CGO_ENABLED=0 GOOS=linux go build -v -o app
docker.io/dkalinin/k8s-simple-app |  ---> Running in 5e21b3183646
docker.io/dkalinin/k8s-simple-app | net
docker.io/dkalinin/k8s-simple-app | net/textproto
docker.io/dkalinin/k8s-simple-app | crypto/x509
docker.io/dkalinin/k8s-simple-app | internal/x/net/http/httpguts
docker.io/dkalinin/k8s-simple-app | internal/x/net/http/httpproxy
docker.io/dkalinin/k8s-simple-app | crypto/tls
docker.io/dkalinin/k8s-simple-app | net/http/httptrace
docker.io/dkalinin/k8s-simple-app | net/http
docker.io/dkalinin/k8s-simple-app | _/src
docker.io/dkalinin/k8s-simple-app | Removing intermediate container 5e21b3183646
docker.io/dkalinin/k8s-simple-app |  ---> bd702bf4a0c4
docker.io/dkalinin/k8s-simple-app | Step 5/8 : FROM scratch as runtime
docker.io/dkalinin/k8s-simple-app |  --->
docker.io/dkalinin/k8s-simple-app | Step 6/8 : COPY --from=builder /src/app .
docker.io/dkalinin/k8s-simple-app |  ---> Using cache
docker.io/dkalinin/k8s-simple-app |  ---> 753b71824c31
docker.io/dkalinin/k8s-simple-app | Step 7/8 : EXPOSE 8080
docker.io/dkalinin/k8s-simple-app |  ---> Using cache
docker.io/dkalinin/k8s-simple-app |  ---> 3c5e4cdbdc38
docker.io/dkalinin/k8s-simple-app | Step 8/8 : ENTRYPOINT ["/app"]
docker.io/dkalinin/k8s-simple-app |  ---> Using cache
docker.io/dkalinin/k8s-simple-app |  ---> f999be3e0d96
docker.io/dkalinin/k8s-simple-app | Successfully built f999be3e0d96
docker.io/dkalinin/k8s-simple-app | Successfully tagged kbld:1557534242219453000-quay-io-eduk8s-labs-sample-app-go
docker.io/dkalinin/k8s-simple-app | Untagged: kbld:1557534242219453000-quay-io-eduk8s-labs-sample-app-go
docker.io/dkalinin/k8s-simple-app | finished build (using Docker)
resolve | final: docker.io/dkalinin/k8s-simple-app -> kbld:quay-io-eduk8s-labs-sample-app-go-sha256-f999be3e0d96c78dc4d4c8330c8de8aff3c91f5e152f021d01cb3cd0e92a1797
@@ update service/simple-app (v1) namespace: lab-getting-started-k14s-w01-s003 @@ 
  ...
 31, 39             value: k14s user
 32     -         image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
     40 +         image: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-7aed69a761f772d74416df18f92db11f547f70a54c9ab02592dc339d1d6bede2
 33, 41           name: simple-app
 33, 33   status:

Changes

Namespace                          Name        Kind        Conds.  Age  Op      Wait to    Rs  Ri
lab-getting-started-k14s-w01-s003  simple-app  Deployment  2/2 t   22m  update  reconcile  ok  -

Op:      0 create, 0 delete, 2 update, 0 noop
Wait to: 2 reconcile, 0 delete, 0 noop
...
```

__NOTE__: As we're using a real cluster, our deployment can't access the image we just built. This command will not finish, so you will need to exist manually.

```execute-1
<ctrl-c>
```

As you can see, the above output shows that kbld received ytt's produced configuration, and used the docker build command to build simple app image, ultimately capturing a specific reference and passing it onto kapp.

If the deploy was successful you could check out application in your browser or via curl to see the updated response, but this is not our case.

It's also worth showing that kbld not only builds images and updates references but also annotates Kubernetes resources with image metadata it collects and makes it quickly accessible for debugging. This may not be that useful during development but comes handy when investigating environment (staging, production, etc.) state.

```
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