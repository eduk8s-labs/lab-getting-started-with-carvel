# Building container images locally

Kubernetes embraced the use of container images to package source code and its dependencies. Container images can be created using tools such as `docker`, `pack` if using Buildpacks, or the Maven `spring-boot:build-image` command if developing Spring Boot applications.

To rebuild a container image when changing application source would typically involve manually running a build using one of these tools, and then pushing the resulting image to an image registry accessible to the Kubernetes cluster so it can be deployed. Any deployment resource would then need to be modified to use the specific version for the new image.

The [kbld](https://carvel.dev/kbld/) tool from Carvel is a small tool that provides a simple way to insert container image building into a deployment workflow. `kbld` looks for images within application configuration (currently it looks for image keys), checks if there is an associated source code definition, and if so triggers a build of the container image using `docker` (or other defined build mechanism), then finally captures the image digests for any built images, and updates configuration with new image references.

As we are going to build our image locally, we first need to pull down the source code:

```
git clone https://github.com/eduk8s-labs/sample-app-go
```

This has created a new folder called `sample-app-go` where you can find the source code for the application.

The configuration file `config-step-3-build-local/build.yml` is a new configuration, which specifies that the image `quay.io/eduk8s-labs/sample-app-go` should be built from source code in the `sample-app-go` sub directory when `kbld` runs. To view the configuration file run:

```
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

```
ytt template -f config-step-3-build-local/ -v hello_msg="carvel user" | kbld -f- | kapp deploy -a simple-app -f- --diff-changes --yes
```

The output should be similar to the following:

```
quay.io/eduk8s-labs/sample-app-go | starting build (using Docker): sample-app-go -> kbld:rand-1629385054678609072-1651822854105-quay-io-eduk8s-labs-sample-app-go
quay.io/eduk8s-labs/sample-app-go | Sending build context to Docker daemon  86.53kB
quay.io/eduk8s-labs/sample-app-go | Step 1/9 : FROM golang:1.12 as builder
quay.io/eduk8s-labs/sample-app-go | 1.12: 
quay.io/eduk8s-labs/sample-app-go | Pulling from library/golang
quay.io/eduk8s-labs/sample-app-go | dc65f448a2e2: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | 346ffb2b67d7: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | dea4ecac934f: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | 8ac92ddf84b3: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | 7ca605383307: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | 8ac92ddf84b3: Waiting
quay.io/eduk8s-labs/sample-app-go | 020f524b99dd: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | 06036b0307c9: Pulling fs layer
quay.io/eduk8s-labs/sample-app-go | 7ca605383307: Waiting
quay.io/eduk8s-labs/sample-app-go | 020f524b99dd: Waiting
quay.io/eduk8s-labs/sample-app-go | 06036b0307c9: Waiting
quay.io/eduk8s-labs/sample-app-go | 346ffb2b67d7: 
quay.io/eduk8s-labs/sample-app-go | Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | 346ffb2b67d7: Download complete
quay.io/eduk8s-labs/sample-app-go | dea4ecac934f: 
quay.io/eduk8s-labs/sample-app-go | Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | dea4ecac934f: 
quay.io/eduk8s-labs/sample-app-go | Download complete
quay.io/eduk8s-labs/sample-app-go | dc65f448a2e2: 
quay.io/eduk8s-labs/sample-app-go | Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | dc65f448a2e2: 
quay.io/eduk8s-labs/sample-app-go | Download complete
quay.io/eduk8s-labs/sample-app-go | 8ac92ddf84b3: Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | 8ac92ddf84b3: Download complete
quay.io/eduk8s-labs/sample-app-go | 7ca605383307: Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | 7ca605383307: Download complete
quay.io/eduk8s-labs/sample-app-go | 06036b0307c9: Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | 06036b0307c9: Download complete
quay.io/eduk8s-labs/sample-app-go | 020f524b99dd: Verifying Checksum
quay.io/eduk8s-labs/sample-app-go | 020f524b99dd: Download complete
quay.io/eduk8s-labs/sample-app-go | dc65f448a2e2: Pull complete
quay.io/eduk8s-labs/sample-app-go | 346ffb2b67d7: Pull complete
quay.io/eduk8s-labs/sample-app-go | dea4ecac934f: 
quay.io/eduk8s-labs/sample-app-go | Pull complete
quay.io/eduk8s-labs/sample-app-go | 8ac92ddf84b3: Pull complete
quay.io/eduk8s-labs/sample-app-go | 7ca605383307: Pull complete
quay.io/eduk8s-labs/sample-app-go | 020f524b99dd: Pull complete
quay.io/eduk8s-labs/sample-app-go | 06036b0307c9: Pull complete
quay.io/eduk8s-labs/sample-app-go | Digest: sha256:d0e79a9c39cdb3d71cc45fec929d1308d50420b79201467ec602b1b80cc314a8
quay.io/eduk8s-labs/sample-app-go | Status: Downloaded newer image for golang:1.12
quay.io/eduk8s-labs/sample-app-go |  ---> ffcaee6f7d8b
quay.io/eduk8s-labs/sample-app-go | Step 2/9 : WORKDIR /src
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go |  ---> Running in 4c244111f0e1
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container 4c244111f0e1
quay.io/eduk8s-labs/sample-app-go |  ---> dcf55a6df197
quay.io/eduk8s-labs/sample-app-go | Step 3/9 : COPY . .
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go |  ---> d4e6bea834bd
quay.io/eduk8s-labs/sample-app-go | Step 4/9 : RUN CGO_ENABLED=0 GOOS=linux go build -v -o app
quay.io/eduk8s-labs/sample-app-go |  ---> Running in 740b7e0f440f
quay.io/eduk8s-labs/sample-app-go | net
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | net/textproto
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | internal/x/net/http/httpproxy
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | crypto/x509
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | internal/x/net/http/httpguts
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | crypto/tls
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | net/http/httptrace
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | net/http
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | _/src
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container 740b7e0f440f
quay.io/eduk8s-labs/sample-app-go |  ---> c9eb1051b332
quay.io/eduk8s-labs/sample-app-go | Step 5/9 : FROM scratch as runtime
quay.io/eduk8s-labs/sample-app-go | 
quay.io/eduk8s-labs/sample-app-go |  ---> 
quay.io/eduk8s-labs/sample-app-go | Step 6/9 : COPY --from=builder /src/app .
quay.io/eduk8s-labs/sample-app-go |  ---> 87abf861379d
quay.io/eduk8s-labs/sample-app-go | Step 7/9 : EXPOSE 8080
quay.io/eduk8s-labs/sample-app-go |  ---> Running in a256ac262992
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container a256ac262992
quay.io/eduk8s-labs/sample-app-go |  ---> 13368d6c0e29
quay.io/eduk8s-labs/sample-app-go | Step 8/9 : USER 1000
quay.io/eduk8s-labs/sample-app-go |  ---> Running in cf4601d387c6
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container cf4601d387c6
quay.io/eduk8s-labs/sample-app-go |  ---> a2699e620aea
quay.io/eduk8s-labs/sample-app-go | Step 9/9 : ENTRYPOINT ["/app"]
quay.io/eduk8s-labs/sample-app-go |  ---> Running in 5232bb64d4ea
quay.io/eduk8s-labs/sample-app-go | Removing intermediate container 5232bb64d4ea
quay.io/eduk8s-labs/sample-app-go |  ---> 2040a07ce5d0
quay.io/eduk8s-labs/sample-app-go | Successfully built 2040a07ce5d0
quay.io/eduk8s-labs/sample-app-go | Successfully tagged kbld:rand-1629385054678609072-1651822854105-quay-io-eduk8s-labs-sample-app-go
quay.io/eduk8s-labs/sample-app-go | Untagged: kbld:rand-1629385054678609072-1651822854105-quay-io-eduk8s-labs-sample-app-go
quay.io/eduk8s-labs/sample-app-go | finished build (using Docker)
resolve | final: quay.io/eduk8s-labs/sample-app-go -> kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5

@@ update deployment/simple-app (apps/v1) namespace: default @@
  ...
  4,  4       deployment.kubernetes.io/revision: "3"
      5 +     kbld.k14s.io/images: |
      6 +       - Metas:
      7 +         - Path: /home/ubuntu/sample-app-go
      8 +           Type: local
      9 +         - Dirty: false
     10 +           RemoteURL: https://github.com/eduk8s-labs/sample-app-go
     11 +           SHA: 338d4d9e5299952a0bdb10d72b873ebbde0a1410
     12 +           Type: git
     13 +         URL: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5
  5, 14     creationTimestamp: "2021-08-18T23:29:52Z"
  6, 15     generation: 8
  ...
103,112   spec:
104     -   replicas: 2
105,113     selector:
106,114       matchLabels:
  ...
119,127             value: carvel user
120     -         image: quay.io/eduk8s-labs/sample-app-go@sha256:5021a23e0c4a4633bfd6c95b13898cffb88a0e67f109d87ec01b4f896f4b4296
    128 +         image: kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5
121,129           name: simple-app
122,130   status:

Changes

Namespace  Name        Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
default    simple-app  Deployment  2/2 t   15h  update  -       reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

2:57:59PM: ---- applying 1 changes [0/1 done] ----
2:58:01PM: update deployment/simple-app (apps/v1) namespace: default
2:58:01PM: ---- waiting on 1 changes [0/1 done] ----
2:58:01PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
2:58:01PM:  ^ Waiting for generation 10 to be observed
2:58:01PM:  L ok: waiting on replicaset/simple-app-77d4cfc7b (apps/v1) namespace: default
2:58:01PM:  L ok: waiting on replicaset/simple-app-688bf88d9b (apps/v1) namespace: default
2:58:01PM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
2:58:01PM:  L ok: waiting on replicaset/simple-app-57d74b4774 (apps/v1) namespace: default
2:58:01PM:  L ongoing: waiting on pod/simple-app-77d4cfc7b-9skkr (v1) namespace: default
2:58:01PM:     ^ Pending: ContainerCreating
2:58:01PM:  L ok: waiting on pod/simple-app-688bf88d9b-nsjxw (v1) namespace: default
2:58:01PM:  L ongoing: waiting on pod/simple-app-688bf88d9b-nlfnr (v1) namespace: default
2:58:01PM:     ^ Deleting
2:58:02PM: ongoing: reconcile deployment/simple-app (apps/v1) namespace: default
2:58:02PM:  ^ Waiting for 1 unavailable replicas
2:58:02PM:  L ok: waiting on replicaset/simple-app-77d4cfc7b (apps/v1) namespace: default
2:58:02PM:  L ok: waiting on replicaset/simple-app-688bf88d9b (apps/v1) namespace: default
2:58:02PM:  L ok: waiting on replicaset/simple-app-677b96597b (apps/v1) namespace: default
2:58:02PM:  L ok: waiting on replicaset/simple-app-57d74b4774 (apps/v1) namespace: default
2:58:02PM:  L ongoing: waiting on pod/simple-app-77d4cfc7b-9skkr (v1) namespace: default
2:58:02PM:     ^ Pending: ErrImagePull (message: rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5": failed to resolve reference "docker.io/library/kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed)
...
```

Note that you will see that the deployment is failing. This is because we haven't configured `kbld` to push the image to an image registry where Kubernetes can pull it from. We will show the required steps to push the image to an image registry in the next section. For now, stop the attempted deployment by interrupting `kapp`.

Press `Ctrl+c` now.

What you can see from the above output though, is how `kbld` triggered a build of the container image using `docker` based on the requirement for the image in the configuration received from `ytt`. Further, `kbld` modified the image reference to include the new image digest before passing the configuration onto `kapp` for deployment.

It's also worth noting that `kbld` not only builds images and updates references but also annotates Kubernetes resources with image metadata it collects and makes it quickly accessible for debugging. This may not be that useful during development but comes in handy when investigating environment (e.g., staging, production) state.

```
kapp inspect -a simple-app --raw --filter-kind Deployment --tty=false | kbld inspect -f-
```

The output from this will be similar to:

```
Images

Image     kbld:quay-io-eduk8s-labs-sample-app-go-sha256-2040a07ce5d0d41559c23103ae5c56b4d64c9f36bb5859d664b1311e329a79b5
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
