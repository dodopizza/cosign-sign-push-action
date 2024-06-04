# Configure Kubernetes cluster

## Requirements

If you still haven't key pairs to sign the images, please read [Key Generation](./docs/key-management.md) article

## Configure Policy controller

Install helm chart from official repository:

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

For more documentation, refer to: [artifacthub.io/packages/helm/sigstore/policy-controller](https://artifacthub.io/packages/helm/sigstore/policy-controller)

## Create Policy

Create policy with public certificate:

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
```

For more documentation and sample policies, refer to: [docs.sigstore.dev/policy-controller/sample-policies](https://docs.sigstore.dev/policy-controller/sample-policies/)

## Configure namespace

The `policy-controller` admission controller will by default only validate resources in namespaces that have chosen to opt-in. This can be done by adding the label `policy.sigstore.dev/include: "true"` to the namespace resource.

```sh
kubectl label namespace my-secure-namespace policy.sigstore.dev/include=true
```
For more documentation, refer to: [docs.sigstore.dev/policy-controller/overview](https://docs.sigstore.dev/policy-controller/overview/)
