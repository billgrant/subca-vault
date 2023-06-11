# Subca-Vault


## This repo shows an example of how to upload a Signed Intermediate cert and private key into the Hashicorp Vault PKI Secrets Engine when Vault did not create the CSR.

Why do we need this? Because the API call `/pki/intermediate/set-signed` does not support uploading a pem bundle.

### Upload the private key

```curl
  curl \
    --header "X-Vault-Token: <token>" \
    --request POST \
    --data @payload.json \
    https://<vault_address>/v1/<path>/keys/import
```

*Path is the mount path of your PKI Secrets Engine the default is `pki`*

`payload.json`

```json
{
   "key_name":"subca-key",
   "pem_bundle": "-----BEGIN PRIVATE KEY-----\n
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQClwKWypRyUNhJJ
...................snip.........................................
sdNC8yQOXhSCTmRYD3otIxj2Vg==\n
-----END PRIVATE KEY-----"
}
```
*`\n` is required as shown above*



### Set the signed Intermediate cert

```curl
curl \
    --header "X-Vault-Token: <token>" \
    --request POST \
    --data "@int_payload.json" \
    https://https://<vault_address>/v1/<path>/intermediate/set-signed 
```

`int_payload.json`

```json
{
   "certificate": "-----BEGIN CERTIFICATE-----\n
MIIFETCCA/mgAwIBAgITHAAAADgV4gn6az5lHgABAAAAODANBgkqhkiG9w0BAQsF
...................snip.........................................
yiqsz/6FZPLTqg31PVa2s1UuD53c26SbVlyTtEDq4QGiZCfNXvhh9T3MJaeZW6gb
uorf2Pc=\n
-----END CERTIFICATE-----"
}
```
*`\n` is required as shown above*

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