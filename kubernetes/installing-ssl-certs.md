# Installing TLS Certificates as a Kubernetes Secret

In this example I install TLS certificates as [Kubernetes secret resources](https://kubernetes.io/docs/concepts/configuration/secret/) into the `ambassador` namespace.

## The Certificate Files

First, gather your certificates. You will need two files:

* A PEM-encoded public key certificate (called `tls.crt` in this example)
* The private key associated with given certificate (called `tls.key` in this example)

## Delete Existing Certificates

If a previous version of the certs already exist, delete them first.

In this example our secret is named `ambassador-certs`.

```bash
kubectl -n ambassador delete secret ambassador-certs
```

## Install the New Certificates

Now we can use the `kubectl create secret tls` command (yes, that whole thing is one particularly obscure command) to install the certificates into the `ambassador` namespace:

```bash
kubectl -n ambassador create secret tls ambassador-certs --key ./tls.key --cert ./tls.crt
```
