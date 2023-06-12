# Subca-Vault

## This repo shows an example of how to upload a Signed Intermediate cert and private key into the Hashicorp Vault PKI Secrets Engine when Vault did not create the CSR.

Why do we need this? Because the API call `/pki/intermediate/set-signed` does not support uploading a pem bundle.

 **Update** Turns out there is an easier way. The above statement below is true, however, the following call `pki/issuers/import/bundle` will allow this type of upload. 
 
See below for the simplified instructions

### Upload the Pem Bundle

```curl
  curl \
    --header "X-Vault-Token: <token>" \
    --request POST \
    --data @payload.json \
    https://<vault_address>/v1/<path>/issuers/import/bundle
```

*Path is the mount path of your PKI Secrets Engine the default is `pki`*

`payload.json`

```json
{
  "pem_bundle": "-----BEGIN PRIVATE KEY-----\n
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7OvA5O2dM4FMu
...............snip.............................................
nTHYV2fMT7ZjHvmBC4d5IGA=\n
-----END PRIVATE KEY-----\n
-----BEGIN CERTIFICATE-----\n
MIIFlzCCBH+gAwIBAgITHAAAADyn2QnrRes6WAACAAAAPDANBgkqhkiG9w0BAQsF
...............sip..............................................
mR4h8BKavfU6L/ksonmEty1Sg7z486X+zCKu93EmgFEuwuK+139Ly/P2gQ==\n
-----END CERTIFICATE-----\n"
}
```
*`\n` is required as shown above*



### Set the default issuer

```curl
  curl \
    --header "X-Vault-Token: <token>" \
    --request POST \
    --data @replace_payload.json \
    https://<vault_address>/v1/<path>/config/issuers
```

`replace_payload.json`

```json
{
  "default": "issuer-id"
}
```


## Confirming it works

### Create a role using the CLI.


```shell
vault write <path>/roles/example-dot-com \
    allowed_domains=example.com \
    allow_subdomains=true max_ttl=72h
```

*Path is the mount path of your PKI Secrets Engine the default is `pki`*

### Now we can issue certficates

```
vault write <path>/issue/example-dot-com \
    common_name=test.example.com
```

*Path is the mount path of your PKI Secrets Engine the default is `pki`*