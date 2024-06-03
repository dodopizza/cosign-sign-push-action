# cosign-sign-push-action

## 1. Features

- Based on `cosign` local keys.
- Signs images and pushes them to a container registry.

## 2. How to Use

### 2.1. Cosign Local Keys

#### Workflow

1. Generate a password for the private key.
2. Generate a key pair.
3. Store the private key and password in GitHub Actions Secrets.

#### Generate Key Pair

You can generate keys using:

1. [Official Binary](https://docs.sigstore.dev/system_config/installation/)

    ```sh
    COSIGN_PASSWORD=<your_private_key_password> cosign generate-key-pair
    ```

2. [Docker Image by VMware](https://hub.docker.com/r/bitnami/cosign/)

    ```sh
    docker run --rm -it \
    -e COSIGN_PASSWORD=<your_private_key_password> \
    -v "$(pwd):/keys" \
    -w /keys \
    bitnami/cosign:latest \
    generate-key-pair
    ```

Default GitHub Action Secrets for keys:

- `COSIGN_PASSWORD`: Password for the private key.
- `COSIGN_PUBLIC_KEY`: Content of the file `cosign.pub`.
- `COSIGN_PRIVATE_KEY`: Content of the file `cosign.key`.

You can generate and store keys directly in GitHub Actions Secrets with the command:

```bash
GITHUB_TOKEN=xxx cosign generate-key-pair github://dodopizza/app
```

**Note:** You can't export the public key with `cosign` from GitHub Action Secrets.

### 2.2. GitHub Action

#### Workflow

1. Set up `cosign` (e.g., `sigstore/cosign-installer`).
2. Log in to the container registry (e.g., `docker/login-action`).
3. Sign the image using this action.

#### Input Variables

| Variable             | Required | Description                                                                                                               |
| -------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `image-tags`         | Yes      | List of image tags. Tags are used to denote different versions or variants of an image, e.g., "latest", "v1.0", "stable". |
| `image-digest`       | Yes      | Image digest. This is a unique identifier for the image, represented as a hash of its contents.                           |
| `cosign-private-key` | Yes      | Cosign private key used for signing container images.                                                                     |
| `cosign-password`    | Yes      | Password for the Cosign private key.                                                                                      |

### 2.3. Configure Kubernetes Cluster

#### Workflow

1. Add the Helm chart, configure values, and deploy the Policy Controller.
2. Create policies.

#### Helm Chart

1. Add the Sigstore Helm repository:

    ```sh
    helm repo add sigstore https://sigstore.github.io/helm-charts
    ```

2. Update your local Helm chart repository cache:

    ```sh
    helm repo update
    ```

3. Install the `policy-controller` chart from the Sigstore repository:

    ```sh
    helm install policy-controller sigstore/policy-controller
    ```

    Using a `values.yaml` file:

    ```sh
    helm install policy-controller sigstore/policy-controller -f values.yaml
    ```

Helm chart documentation: [artifacthub.io/packages/helm/sigstore/policy-controller](https://artifacthub.io/packages/helm/sigstore/policy-controller)

#### Create Policies

Sample policy:

```yaml
apiVersion: policy.sigstore.dev/v1alpha1
kind: ClusterImagePolicy
metadata:
  name: custom-key-attestation-sbom-spdxjson
spec:
  images:
  - glob: "**"
  authorities:
  - name: custom-key
    key:
      data: |
        -----BEGIN PUBLIC KEY-----
        MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEOc6HkISHzVdUbtUsdjYtPuyPYBeg
        4FCemyVurIM4KEORQk4OAu8ZNwxvGSoY3eAabYaFIPPQ8ROAjrbdPwNdJw==
        -----END PUBLIC KEY-----
    attestations:
    - name: must-have-spdxjson
      predicateType: https://spdx.dev/Document
      policy:
        type: cue
        data: |
          predicateType: "https://spdx.dev/Document"
```

For more documentation and sample policies, refer to: [docs.sigstore.dev/policy-controller/sample-policies](https://docs.sigstore.dev/policy-controller/sample-policies/)

## 3. Usage example:

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
- [Helm Chart for Policy Controller](https://artifacthub.io/packages/helm/sigstore/policy-controller)
