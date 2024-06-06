# cosign-sign-push-action

## About

GitHub Action to sign images using local `cosign` keys; see the [Cosign documentation on local keys][8] for more details.

## Getting started

1. **Generate cosign keys**:
   - To use this action, you need to start by generating cosign keys. Follow the [key generation][4] guide for detailed instructions.
2. **(Optional) Set up image verification in Kubernetes**:
   - If you want to enforce image verification in your Kubernetes cluster, you can set up a policy controller. Here are a few options to consider:
     - **Cosign policy controller**: our example configuration includes a Cosign policy and instructions on how to configure a namespace in your Kubernetes cluster. For more details, refer to the [install Cosign policy controller][1] guide and the [Cosign policy controller documentation][7].
     - **Kyverno policies**: use [Kyverno policies][5] to enforce image signature verification.
     - **OPA Gatekeeper**: use [OPA Gatekeeper][6] with custom policies for image verification.

## Reference links

### Official documentation
- [Cosign][2]
- [Cosign local keys][8]
- [Cosign installer for GitHub Actions][3]
- [Cosign policy controller][7]
- [Kyverno policies][5]
- [OPA Gatekeeper][6]

### Quick start guides
- [Key generation][4]
- [Install Cosign policy controller][1]

## Input variables

| Variable             | Required | Description                                                                                                               |
| -------------------- | :------: | ------------------------------------------------------------------------------------------------------------------------- |
| `image-tags`         |   Yes    | List of image tags. Tags are used to denote different versions or variants of an image, e.g., "latest", "v1.0", "stable". |
| `image-digest`       |   Yes    | Image digest. This is a unique identifier for the image, represented as a hash of its contents.                           |
| `cosign-private-key` |   Yes    | Cosign private key used for signing container images.                                                                     |
| `cosign-password`    |   Yes    | Password for the Cosign private key.                                                                                      |

## Usage example

To use this example, you need to generate Cosign keys and store them in GitHub Actions secrets:

- `COSIGN_PASSWORD`: password for the private key.
- `COSIGN_PRIVATE_KEY`: private key.

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
    env:
      IMAGE_NAME: ghcr.io/dodopizza/app
      IMAGE_TAG: latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

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

    - name: Build and Push container images
      uses: docker/build-push-action@v5
      id: build-and-push
      with:
        file: ./docker-image/Dockerfile
        platforms: linux/amd64,linux/arm/v7,linux/arm64
        push: true
        provenance: false
        tags: ${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}

    - name: Sign container images with a key
      uses: dodopizza/cosign-sign-push-action@0.0.7
      with:
        image-tags: ${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
        image-digest: ${{ steps.build-and-push.outputs.digest }}
        cosign-private-key: ${{ secrets.COSIGN_PRIVATE_KEY }}
        cosign-password: ${{ secrets.COSIGN_PASSWORD }}

    - name: Output image tags
      env:
        TAGS: ${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
      run: |
        echo "## Built images with the following tags" >>$GITHUB_STEP_SUMMARY
        echo "${TAGS}" >>$GITHUB_STEP_SUMMARY
```

[1]: ./docs/k8s.md
[2]: https://docs.sigstore.dev/
[3]: https://github.com/sigstore/cosign-installer/
[4]: ./docs/key-management.md
[5]: https://kyverno.io/policies/
[6]: https://github.com/open-policy-agent/gatekeeper
[7]: https://docs.sigstore.dev/cosign/policy_controller/
[8]: https://docs.sigstore.dev/cosign/local_keys/
