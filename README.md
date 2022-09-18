# 📦 space-unlimited-space 📦

This repository is a playground to demostrate how you can use [Github Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) to store **arbitrary artifacts** and retrieve them without authentication.

This is the similar to the mechanism used by [Brew](https://brew.sh/) to store and distribute artifacts.

## Requirements

You will need to have `skopeo` installed with:

```
brew install skopeo
```

[More info on skopeo here](https://github.com/containers/skopeo).

## Publishing

### 1. Logging in with Github Container Registry

First, make sure you're logged in with Github Container Registry:

```
skopeo login ghcr.io
```

Username: Your Github Username
Password: A Github Personal Access Token (PAT) with `write:packages`, `read:packages` and `delete:packages` permission. You can create it at: [https://github.com/settings/tokens](https://github.com/settings/tokens).

### 2. Uploading Image to Github Container Registry

The folder `anakin` in this repo contains a valid OCI Image.

You can upload the image with:

```
skopeo copy oci:<containername>:<tag> docker://ghcr.io/<user>/<repo>/<containername>:<tag>
```

Like the following:

```
$ skopeo copy oci:anakin:1.0.0 docker://ghcr.io/cortinico/space-unlimited-space/anakin:1.0.0
Getting image source signatures
Copying blob 57b8eee16897 done
Copying config d7a38a28d7 done
Writing manifest to image destination
Storing signatures
```

### 3. Inspect the Container Information

You can use `skopeo inspect` to query the available tarballs:

```
$ skopeo inspect docker://ghcr.io/cortinico/space-unlimited-space/anakin:1.0.0

{
    "Name": "ghcr.io/cortinico/space-unlimited-space/anakin",
    "Digest": "sha256:aa986f18e5556f9ce7ce265242ec6457ea2802030d92e5dc4c3c807a495db1e4",
    "RepoTags": [
        "1.0.0"
    ],
    "Created": null,
    "DockerVersion": "",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "darwin",
    "Layers": [
        "sha256:57b8eee16897b0bb1d00190e99fb722dbaa32ecd1eb70686012ecf814ce329ba"
    ],
    "Env": null
}
```

## Layout

The container has to follow the [OCI specs](https://github.com/opencontainers/image-spec):

```
anakin/
├── blobs
│   └── sha256
│       ├── 57b8eee16897b0bb1d00190e99fb722dbaa32ecd1eb70686012ecf814ce329ba
│       ├── aa986f18e5556f9ce7ce265242ec6457ea2802030d92e5dc4c3c807a495db1e4
│       └── d7a38a28d7b1cf81b95ee50a4d011871378018ae5ee366acc5505273fee5a0b3
├── index.json
└── oci-layout
```

Specifically:
* `oci-layout` is immutable
* `index.json` is the [Image Index](https://github.com/opencontainers/image-spec/blob/main/image-index.md). Contains a reference to the Image Manifest.
* `blobs/sha256` is the folder where blobs are stores. Files should be named using their own SHA256 (or other algos).
* `57b8eee16897b0bb1d00190e99fb722dbaa32ecd1eb70686012ecf814ce329ba` is a tarball (it's a `.tar.gz` that contains an image)
* `aa986f18e5556f9ce7ce265242ec6457ea2802030d92e5dc4c3c807a495db1e4` is the [Image Manifest](https://github.com/opencontainers/image-spec/blob/main/manifest.md) contains references to the Layers (the Tarballs).
* `d7a38a28d7b1cf81b95ee50a4d011871378018ae5ee366acc5505273fee5a0b3` is the [Image Config](https://github.com/opencontainers/image-spec/blob/main/config.md). This is immutable as we're not doing filesystem operation.

## Consuming via HTTP

You can download a public manifest from Github Container Registry with:

```
curl -L --verbose \
   --header "Accept: application/vnd.oci.image.index.v1+json" \
   --header "Authorization: Bearer QQ==" \
   https://ghcr.io/v2/cortinico/space-unlimited-space/anakin/manifests/1.0.0 \
   -o manifest.json
```

To download a public tarball instead:

```
curl -L --verbose \
   --header "Authorization: Bearer QQ==" \
   https://ghcr.io/v2/cortinico/space-unlimited-space/blobs/sha256:57b8eee16897b0bb1d00190e99fb722dbaa32ecd1eb70686012ecf814ce329ba \
    -o output.tar.gz
```

## Useful commands

Compute the SHA256 of all the files:

```
shasum -a 256 anakin/blobs/sha256/*
```

List filesize of all the files:

```
ls -la anakin/blobs/sha256/*
```