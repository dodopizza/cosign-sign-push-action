# Key Generation

You can generate keys using:

1. [Official Binary](https://docs.sigstore.dev/system_config/installation/)

    - Save keys in local files

        ```sh
        COSIGN_PASSWORD=<your_private_key_password> cosign generate-key-pair
        ```

        Output files:

        - `cosign.pub`: public key.
        - `cosign.key`: private key.

    - Save keys in GitHub Action Secrets

        ```bash
        GITHUB_TOKEN=xxx cosign generate-key-pair github://dodopizza/app
        ```

        Output GitHub Action Secrets for keys:

        - `COSIGN_PASSWORD`: password for the private key.
        - `COSIGN_PUBLIC_KEY`: public key.
        - `COSIGN_PRIVATE_KEY`: private key.

        **Note:** You can't export the public key with `cosign` from GitHub Action Secrets.

2. [Docker Image by VMware](https://hub.docker.com/r/bitnami/cosign/)

    ```sh
    docker run --rm -it \
    -e COSIGN_PASSWORD=<your_private_key_password> \
    -v "$(pwd):/keys" \
    -w /keys \
    bitnami/cosign:latest \
    generate-key-pair
    ```
For more documentation and sample policies, refer to: [docs.sigstore.dev/key_management](https://docs.sigstore.dev/key_management/)
