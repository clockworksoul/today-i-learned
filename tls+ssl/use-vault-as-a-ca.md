# Use Hashicorp Vault to Build Your Own Certificate Authority (CA)

_Source: https://learn.hashicorp.com/vault/secrets-management/sm-pki-engine_

Vault's PKI secrets engine can dynamically generate X.509 certificates on demand. This allows services to acquire certificates without going through the usual manual process of generating a private key and Certificate Signing Request (CSR), submitting to a CA, and then waiting for the verification and signing process to complete.

### Challenge

Organizations should protect their website; however, the Traditional PKI process workflow takes a long time which motivates organizations to create certificates which do not expire for a year or more.

### Solution

Use Vault to create X509 certificates for usage in MTLS or other arbitrary PKI encryption. While this can be used to create web server certificates. If users do not import the CA chains, the browser will complain about self-signed certificates.

Creating PKI certificates is generally a cumbersome process using traditional tools like openssl or even more advanced frameworks like CFSSL. These tools also require a human component to verify certificate distribution meets organizational security policies.

Vault PKI secrets engine makes this a lot simpler. The PKI secrets engine can be an Intermediate-Only certificate authority which potentially allows for higher levels of security.

1. Store CA outside the Vault (air gapped)
2. Create CSRs for the intermediates
3. Sign CSR outside Vault and import intermediate
4. Issue leaf certificates from the Intermediate CA

## Prerequisites

To perform the tasks described in this guide, you need to have a Vault environment. Refer to the [Getting Started](https://learn.hashicorp.com/vault/getting-started/install) guide to install Vault.

Or you can use the [Vault Playground](https://www.katacoda.com/hashicorp/scenarios/vault-playground) environment.

### Policy requirements

NOTE: For the purpose of this guide, you can use `root` token to work with Vault. However, it is recommended that root tokens are only used for just enough initial setup or in emergencies. As a best practice, use tokens with appropriate set of policies based on your role in the organization.

To perform all tasks demonstrated in this guide, your policy must include the following permissions:

```
# Enable secrets engine
path "sys/mounts/*" {
  capabilities = [ "create", "read", "update", "delete", "list" ]
}

# List enabled secrets engine
path "sys/mounts" {
  capabilities = [ "read", "list" ]
}

# Work with pki secrets engine
path "pki*" {
  capabilities = [ "create", "read", "update", "delete", "list", "sudo" ]
}
```

If you are not familiar with policies, complete the policies guide.

## Scenario Introduction

In this guide, you are going to first generate a self-signed root certificate. Then you are going to generate an intermediate certificate which is signed by the root. Finally, you are going to generate a certificate for the `test.example.com` domain.

Each step will be illustrated in using the CLI commands. See the original source for API calls with cURL or the Web UI.

![Overview](use-vault-as-a-ca.png)

In this guide, you will perform the following:

1. Generate Root CA
2. Generate Intermediate CA
3. Create a Role
4. Request Certificates
5. Revoke Certificates
6. Remove Expired Certificates

## Step 1: Generate Root CA

In this step, you are going to generate a self-signed root certificate using PKI secrets engine.

1. First, enable the `pki` secrets engine at the `pki` path.

```bash
$ vault secrets enable pki
```

2. Tune the `pki` secrets engine to issue certificates with a maximum time-to-live (TTL) of 87600 hours.

```bash
$ vault secrets tune -max-lease-ttl=87600h pki
```

3. Generate the _root_ certificate and save the certificate in CA_cert.crt.

```bash
$ vault write -field=certificate pki/root/generate/internal \
        common_name="example.com" \
        ttl=87600h > CA_cert.crt
```

This generates a new self-signed CA certificate and private key. Vault will _automatically_ revoke the generated root at the end of its lease period (TTL); the CA certificate will sign its own Certificate Revocation List (CRL).

4. Configure the CA and CRL URLs.

```bash
$ vault write pki/config/urls \
        issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
        crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"
```

## Step 2: Generate Intermediate CA

Now, you are going to create an intermediate CA using the root CA you regenerated in the previous step.

1. First, enable the pki secrets engine at the pki_int path.

```bash
$ vault secrets enable -path=pki_int pki
```

2. Tune the `pki_int` secrets engine to issue certificates with a maximum time-to-live (TTL) of 43800 hours.

```bash
$ vault secrets tune -max-lease-ttl=43800h pki_int
```

3. Execute the following command to generate an intermediate and save the CSR as `pki_intermediate.csr`.

```bash
$ vault write -format=json pki_int/intermediate/generate/internal \
        common_name="example.com Intermediate Authority" \
        | jq -r '.data.csr' > pki_intermediate.csr
```

4. Sign the intermediate certificate with the root certificate and save the generated certificate as `intermediate.cert.pem`.

```bash
$ vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr \
        format=pem_bundle ttl="43800h" \
        | jq -r '.data.certificate' > intermediate.cert.pem
```

5. Once the CSR is signed and the root CA returns a certificate, it can be imported back into Vault.

```bash
$ vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem
```

## Step 3: Create a Role

A role is a logical name that maps to a policy used to generate those credentials. It allows [configuration parameters](https://www.vaultproject.io/api-docs/secret/pki#create-update-role) to control certificate common names, alternate names, the key uses that they are valid for, and more.

Here are a few noteworthy parameters:

| Param                | Description   |
| -------------------- |---------------|
| `allowed_domains`    | Specifies the domains of the role (used with allow_bare_domains and allow-subdomains options) |
| `allow_bare_domains` | Specifies if clients can request certificates matching the value of the actual domains themselves |
| `allow_subdomains`   | Specifies if clients can request certificates with CNs that are subdomains of the CNs allowed by the other role options (NOTE: This includes wildcard subdomains.) |
| `allow_glob_domains` | Allows names specified in allowed_domains to contain glob patterns (e.g. ftp*.example.com) |

In this step, you are going to create a role named `example-dot-com`.

Create a role named `example-dot-com` which allows subdomains.

```bash
$ vault write pki_int/roles/example-dot-com \
        allowed_domains="example.com" \
        allow_subdomains=true \
        max_ttl="720h"
```

## Step 4: Request Certificates

Keep certificate lifetimes short to align with Vault's philosophy of short-lived secrets.

Execute the following command to request a new certificate for the `test.example.com` domain based on the `example-dot-com` role.

```bash
$ vault write pki_int/issue/example-dot-com common_name="test.example.com" ttl="24h"

Key                 Value
---                 -----
certificate         -----BEGIN CERTIFICATE-----
MIIDwzCCAqugAwIBAgIUTQABMCAsXjG6ExFTX8201xKVH4IwDQYJKoZIhvcNAQEL
BQAwGjEYMBYGA1UEAxMPd3d3LmV4YW1wbGUuY29tMB4XDTE4MDcyNDIxMTMxOVoX
             ...

-----END CERTIFICATE-----
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDQTCCAimgAwIBAgIUbMYp39mdj7dKX033ZjK18rx05x8wDQYJKoZIhvcNAQEL
             ...

-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAte1fqy2Ekj+EFqKV6N5QJlBgMo/U4IIxwLZI6a87yAC/rDhm
W58liadXrwjzRgWeqVOoCRr/B5JnRLbyIKBVp6MMFwZVkynEPzDmy0ynuomSfJkM
             ...

-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       4d:00:01:30:20:2c:5e:31:ba:13:11:53:5f:cd:b4:d7:12:95:1f:82
```

The response contains the PEM-encoded private key, key type and certificate serial number.

## Step 5: Revoke Certificates

If a certificate must be revoked, you can easily perform the revocation action which will cause the CRL to be regenerated. When the CRL is regenerated, any expired certificates are removed from the CRL.

In certain circumstances, you may wish to revoke an issued certificate.

To revoke a certificate, execute the following command.

```bash
$ vault write pki_int/revoke serial_number=<serial_number>
```

Example:

```bash
$ vault write pki_int/revoke \
        serial_number="48:97:82:dd:f0:d3:d9:7e:53:25:ba:fd:f6:77:3e:89:e5:65:cc:e7"
Key                        Value
---                        -----
revocation_time            1532539632
revocation_time_rfc3339    2018-07-25T17:27:12.165206399Z
```

## Step 6: Remove Expired Certificates

Keep the storage backend and CRL by periodically removing certificates that have expired and are past a certain buffer period beyond their expiration time.

To remove revoked certificate and clean the CRL.

```bash
$ vault write pki_int/tidy tidy_cert_store=true tidy_revoked_certs=true
```
