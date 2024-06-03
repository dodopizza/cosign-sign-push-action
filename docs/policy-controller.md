
# Configure Policy controller in kubernetes cluster

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

Helm chart documentation: [artifacthub.io/packages/helm/sigstore/policy-controller](https://artifacthub.io/packages/helm/sigstore/policy-controller)
