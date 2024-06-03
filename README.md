# cosign-sign-push-action

## Features

- Based on `cosign` local keys.
- Signs images and pushes them to a container registry.

## Topics

- [Key Generation](./docs/key-management.md)
- [Configure Policy controller in kubernetes cluster](./docs/policy-controller.md)
- [Create Policies](./docs/policy.md)

## Input Variables

| Variable             | Required | Description                                                                                                               |
| -------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `image-tags`         | Yes      | List of image tags. Tags are used to denote different versions or variants of an image, e.g., "latest", "v1.0", "stable". |
| `image-digest`       | Yes      | Image digest. This is a unique identifier for the image, represented as a hash of its contents.                           |
| `cosign-private-key` | Yes      | Cosign private key used for signing container images.                                                                     |
| `cosign-password`    | Yes      | Password for the Cosign private key.                                                                                      |

## Usage example:

```yaml
name: Build and sign image
on:
  push:
    branches:
    - main

jobs:
  build-image:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    name: build-image
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 1

    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Set up Cosign
      uses: sigstore/cosign-installer@v3.5.0

    - name: Login to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Set the image metadata
      id: docker-meta
      uses: docker/metadata-action@v5
      with:
        images: ghcr.io/dodopizza/app
        tags: type=raw,value={{branch}}-{{sha}}

    - name: Build and Push container images
      uses: docker/build-push-action@v5
      id: build-and-push
      with:
        file: ./docker-image/Dockerfile
        platforms: linux/amd64,linux/arm/v7,linux/arm64
        push: true
        provenance: false
        tags: ${{ steps.docker-meta.outputs.tags }}

    - name: Sign with a key and Push container images
      uses: dodopizza/cosign-sign-push-action@0.0.7
      with:
        image-tags: ${{ steps.docker-meta.outputs.tags }}
        image-digest: ${{ steps.build-and-push.outputs.digest }}
        cosign-private-key: ${{ secrets.COSIGN_PRIVATE_KEY }}
        cosign-password: ${{ secrets.COSIGN_PASSWORD }}

    - name: Output image tags
      env:
        TAGS: ${{ steps.docker-meta.outputs.tags }}
      run: |
        echo "## Built images with the following tags" >>$GITHUB_STEP_SUMMARY
        images=""
        for tag in ${TAGS}; do
          echo "${tag}" >>$GITHUB_STEP_SUMMARY
        done
```

## Reference links

- [Cosign Documentation](https://docs.sigstore.dev/)
- [Cosign Installer GitHub](https://github.com/sigstore/cosign-installer/)
