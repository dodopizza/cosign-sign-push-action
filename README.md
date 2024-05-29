# cosign-sign-push-action

## Features

- sign image with cosign local key and push it to container registry

## Documentation

- [Cosign](https://docs.sigstore.dev/)
- [Cosign installer](https://github.com/sigstore/cosign-installer/)

## Cosign keys

First, you need to create/generate a password for private key. Then generate keys:

1. With docker

    ```bash
    docker run --rm -it \
    -e COSIGN_PASSWORD=<your_private_key_password> \
    -v "$(pwd):/keys" \
    -w /keys \
    bitnami/cosign:latest \
    generate-key-pair
    ```

2. With binary:

    ```bash
    COSIGN_PASSWORD=<your_private_key_password> cosign generate-key-pair
    ```

Store keys in GitHub Action Secrets:

- COSIGN_PASSWORD - a password for private key
- COSIGN_PUBLIC_KEY - content of the file cosign.pub
- COSIGN_PRIVATE_KEY - content of the file cosign.key

You can generate and store keys directly in GitHub Actions Secrets with command:

```bash
GITHUB_TOKEN=xxx cosign generate-key-pair github://dodopizza/app
```

But remember, you can't export public key with cosign from GitHub Action Secrets. 

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