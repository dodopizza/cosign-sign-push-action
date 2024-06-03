# Create Policies

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
