# Sign the image using Notation

This section instructs the necessary steps to sign a pause container image using Notation.

1. Pull the `pause` image.
```sh
docker pull registry.k8s.io/pause:3.9
```

2. Re-tag the pause container image.
```sh
docker tag registry.k8s.io/pause:3.9 ghcr.io/kubeconchina2023/test-verify-image:signed                                   
docker push ghcr.io/kubeconchina2023/test-verify-image:signed 
```

3. Generate a key and certificate to sign the image.
```sh
notation cert generate-test --default "kubeconchina2023.io" 
```

4. Sign the image.
```sh
notation sign ghcr.io/kubeconchina2023/test-verify-image:signed
```

5. Use the ORAS CLI to query the container registry for supply chain artifacts associated with a specific container image.
```sh
oras discover ghcr.io/kubeconchina2023/test-verify-image:signed -o tree
ghcr.io/kubeconchina2023/test-verify-image@sha256:0fc1f3b764be56f7c881a69cbd553ae25a2b5523c6901fbacb8270307c29d0c4
└── application/vnd.cncf.notary.signature
    └── sha256:f7d58c03d403411d8b51e33b8670aab97f0d14daad3851366c1632b954553053
```

# Verify the image signature using Notation

1. Add the generated certificate from above to verify the image.

Note that I’m on a Mac so the PATH of this cert file may change depending on your platform. You can inspect it by `notation cert ls` command.

```sh
notation cert add --type ca --store kubeconchina2023 /Users/<USER>/Library/Application\ Support/notation/truststore/x509/ca/kubeconchina2023.io/kubeconchina2023.io.crt 
```

2. Create a trust policy.
```sh
cat <<EOF > ./trustpolicy.json
{
    "version": "1.0",
    "trustPolicies": [
        {
            "name": "kubeconchina2023",
            "registryScopes": [ "*" ],
            "signatureVerification": {
                "level" : "strict"
            },
            "trustStores": [ "ca:kubeconchina2023.io" ],
            "trustedIdentities": [
                "*"
            ]
        }
    ]
}
EOF
```

4. Import the trust policy.
```sh
notation policy import ./trustpolicy.json
```

5. Verify the image signature.
```sh
notation verify ghcr.io/kubeconchina2023/test-verify-image:signed        
Warning: Always verify the artifact using digest(@sha256:...) rather than a tag(:signed) because resolved digest may not point to the same signed artifact, as tags are mutable.
Successfully verified signature for ghcr.io/kubeconchina2023/test-verify-image@sha256:0fc1f3b764be56f7c881a69cbd553ae25a2b5523c6901fbacb8270307c29d0c4
```