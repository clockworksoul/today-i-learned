# Dissecting a TLS/SSL Certificate

_Sources: [[1]](https://jvns.ca/blog/2017/01/31/whats-tls/), [[2]](https://en.wikipedia.org/wiki/Transport_Layer_Security), [[3]](https://en.wikipedia.org/wiki/X.509)_

Yes, I already knew most of this, but [this article](https://jvns.ca/blog/2017/01/31/whats-tls/) inspired me to collect it all in one place for posterity. Plus, I always have to look up those `openssl` commands.

## What are SSL and TLS?

Transport Layer Security (TLS), and its now-deprecated predecessor, Secure Sockets Layer (SSL), are cryptographic protocols designed to provide communications security over a computer network.

There are several versions of these protocols, but in a nutshell newer versions of SSL are called TLS (the version after SSL 3.0 is TLS 1.0).

| Protocol | Published   |  Status                       |
| -------- |-------------|-------------------------------|
| SSL 1.0  | Unpublished | Unpublished                   |
| SSL 2.0  | 1995        | Deprecated in 2011 (RFC 6176) |
| SSL 3.0  | 1996        | Deprecated in 2015 (RFC 7568) |
| TLS 1.0  | 1999        | Deprecated in 2020            |
| TLS 1.1  | 2006        | Deprecated in 2020            |
| TLS 1.2  | 2008        |                               |
| TLS 1.3  | 2018        |                               |

## What's a Certificate?

Suppose I'm checking my email at `https://mail.google.com`

`mail.google.com` is running a HTTPS server on port 443. I want to make sure that I'm **actually** talking to `mail.google.com` and not some other random server on the internet owned by EVIL PEOPLE.

So, let's start by using `openssl` to look at `mail.google.com`'s certificate: `openssl s_client -connect mail.google.com:443`

This is going to print a bunch of stuff, but we'll just focus on the certificate:

```bash
$ openssl s_client -connect mail.google.com:443

...

-----BEGIN CERTIFICATE-----
MIIFoTCCBImgAwIBAgIRAPv4g2bpCUCICAAAAAA9N2AwDQYJKoZIhvcNAQELBQAw
QjELMAkGA1UEBhMCVVMxHjAcBgNVBAoTFUdvb2dsZSBUcnVzdCBTZXJ2aWNlczET
MBEGA1UEAxMKR1RTIENBIDFPMTAeFw0yMDA0MjgwNzM4MzhaFw0yMDA3MjEwNzM4
MzhaMGkxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQH
Ew1Nb3VudGFpbiBWaWV3MRMwEQYDVQQKEwpHb29nbGUgTExDMRgwFgYDVQQDEw9t
YWlsLmdvb2dsZS5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC/
Vh6R70i2kH+4L4PMJoeq2U+RTs/KVDeb9VEoIOnJsvjgrkMNjd7+Eedi5kJCA2YQ
orwXneg/CcBZ6QyO162spibTg3GJ2cp5DurHmjzSE7g+Wh5zBeQ3QkoKfGhoBH7/
45zCDh9MbsbvD4n7VLGtY1UtOSZ4PUtwWNZqUP088PcnAqsauUqSJEM/IbPDGnQC
shTj6f+SGy5VJP/EIo7VjtEAkExLSkUxjHA4v5qBrdqaZBCtu929JDzyi99CJX/h
tZNYqwNTNnT4Ck6Aeb9C6O0zCdYwWFYy8mQFMR9dWJhfqmS3uH/1nydOdJ0TomLU
9qIlEvno9ezYpQNpQ3V1AgMBAAGjggJpMIICZTAOBgNVHQ8BAf8EBAMCBaAwEwYD
VR0lBAwwCgYIKwYBBQUHAwEwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQUs5UKtiHr
TgXXPIBncjedM/YTbuwwHwYDVR0jBBgwFoAUmNH4bhDrz5vsYJ8YkBug630J/Ssw
ZAYIKwYBBQUHAQEEWDBWMCcGCCsGAQUFBzABhhtodHRwOi8vb2NzcC5wa2kuZ29v
Zy9ndHMxbzEwKwYIKwYBBQUHMAKGH2h0dHA6Ly9wa2kuZ29vZy9nc3IyL0dUUzFP
MS5jcnQwLAYDVR0RBCUwI4IPbWFpbC5nb29nbGUuY29tghBpbmJveC5nb29nbGUu
Y29tMCEGA1UdIAQaMBgwCAYGZ4EMAQICMAwGCisGAQQB1nkCBQMwLwYDVR0fBCgw
JjAkoCKgIIYeaHR0cDovL2NybC5wa2kuZ29vZy9HVFMxTzEuY3JsMIIBBgYKKwYB
BAHWeQIEAgSB9wSB9ADyAHcAsh4FzIuizYogTodm+Su5iiUgZ2va+nDnsklTLe+L
kF4AAAFxv/AlWQAABAMASDBGAiEApsxkN5iBlmXYyD2y880uDUXwP1lophOidTdY
379yOh4CIQDPKU+xij8S5iZeJXHQF3cKaPyGOTncaufuutOWhzUC5gB3AF6nc/nf
VsDntTZIfdBJ4DJ6kZoMhKESEoQYdZaBcUVYAAABcb/wJXgAAAQDAEgwRgIhALF7
sFGN3YIkG2J7t2Vj7zjr7O2CDEXe57LPADdzf3lvAiEAkuYej2F4MO7PyoWzOas0
zyaXOy+ae8dmFP2fbVm2yG0wDQYJKoZIhvcNAQELBQADggEBAJP3SOUtKRqCa2da
51FRgTm9voyohA7g67QjumDEXrgZJSY+fn7tHG7ggo3xfVMpcFdOGe8ZzIFOs1C1
b+F2gEaDLKTYOUZgX4UtQZG8REWLXbqaj6E+Jck1+18Oz+m4gdy7h8Qb/YQ6ET2a
BdHBYeV9SsMhhys5i6mwwVfLdbeHeitTNo88riHfV6bYO1AbJ1xFpNi9PJWSXkhV
V30oox8QVfec6isUGrbiph0MbwN5eSasyRr4HcoWM6nntzabVk3m1leFb7UVMlwv
0whXtscmqsMHTtv/rjBr7Nxk4aTho1OFYE7+P4N+uI99OngaO0XgOFLRO7VooHGg
DAmf19A=
-----END CERTIFICATE-----

...
```

We're looking at [X.509](https://en.wikipedia.org/wiki/X.509), a standard format for describing public key certificates.

Our next mission is to parse this certificate. We do that by piping the data to `openssl x509`:

```bash
$ openssl s_client -connect mail.google.com:443 | openssl x509 -in /dev/stdin -text -noout

depth=2 OU = GlobalSign Root CA - R2, O = GlobalSign, CN = GlobalSign
verify return:1
depth=1 C = US, O = Google Trust Services, CN = GTS CA 1O1
verify return:1
depth=0 C = US, ST = California, L = Mountain View, O = Google LLC, CN = mail.google.com
verify return:1
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            fb:f8:83:66:e9:09:40:88:08:00:00:00:00:3d:37:60
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=Google Trust Services, CN=GTS CA 1O1
        Validity
            Not Before: Apr 28 07:38:38 2020 GMT
            Not After : Jul 21 07:38:38 2020 GMT
        Subject: C=US, ST=California, L=Mountain View, O=Google LLC, CN=mail.google.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:bf:56:1e:91:ef:48:b6:90:7f:b8:2f:83:cc:26:
                    87:aa:d9:4f:91:4e:cf:ca:54:37:9b:f5:51:28:20:
                    ... <snip> ...
                    62:d4:f6:a2:25:12:f9:e8:f5:ec:d8:a5:03:69:43:
                    75:75
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            ... <snip> ...

            X509v3 Subject Alternative Name: 
                DNS:mail.google.com, DNS:inbox.google.com
    Signature Algorithm: sha256WithRSAEncryption
         93:f7:48:e5:2d:29:1a:82:6b:67:5a:e7:51:51:81:39:bd:be:
         8c:a8:84:0e:e0:eb:b4:23:ba:60:c4:5e:b8:19:25:26:3e:7e:
         ... <snip> ...
         8f:7d:3a:78:1a:3b:45:e0:38:52:d1:3b:b5:68:a0:71:a0:0c:
         09:9f:d7:d0
```

Some of the most useful bits (to us) are:

* The `Subject` section contains the "common name": `CN=mail.google.com`. Counterintuitively you should ignore this field and look at the "subject alternative name" field instead.
* The `Validity` section includes a valid date range with an expiration date. This example expires Jul 21 07:38:38 2020 GMT.
* The `X509v3 Subject Alternative Name` section has the list of domains that this certificate works for. This is `mail.google.com` and `inbox.google.com`.
* The `Subject Public Key Info` section tells us the **public key** that we're going to use to communicate with `mail.google.com`. [Public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) is fascinating, but it's outside the scope of this summary.
* The signature, which can be used to verify the authenticity of the certificate. Without this, anybody could simply issue a certificate for any domain they wanted!

## Certificate Signing

Every certificate on the internet is basically two parts put together:

1. The certificate proper, containing the public key, the domain(s) and dates it's valid for, and so on.
2. A signature by some trusted authority. Essentially a cryptographic stamp of authenticity.

Most OS distributions include a number of certificates -- public [CA certificates](https://en.wikipedia.org/wiki/Certificate_authority) -- that computer trusts to sign other certificates. In Linux these can be found in `/etc/ssl/certs`. On Mac these are installed in the OS X Keychain and harder to find. Aside: to install these in a Linux Docker container, you'll need to do `apt-get install ca-certificates`.

On my computer I have 267 of these certificates:

```bash
$ ls -1 /etc/ssl/certs/ | wc -l
267
```

If any one of them signed a `mail.google.com` certificate, my computer would accept it as authentic. However, if some random person -- who's public key won't be among the 267 trusted entities on my computer -- signed a certificate, my computer would reject the certificate.

The `mail.google.com` certificate is:

* `C = US, ST = California, L = Mountain View, O = Google LLC, CN = mail.google.com`
* Which is signed by a "Google Trust Services" certificate
* Which is signed by a "GlobalSign Root CA - R2" certificate

I have an `GlobalSign_Root_CA_-_R2.pem` file on my computer, which is probably why my computer trusts this certificate.

## What Does Getting a Certificate Issued Look Like?

So when you get a certificate issued, basically how it works is:

1. You generate a [public/private key pair](https://en.wikipedia.org/wiki/Public-key_cryptography) for your certificate:

    1. The public key will contain your domain, expiration, and other public information.
    2. The private key for your certificate is exactly that. Keep it safe. You'll use this key every time you establish an SSL connection.

2. You pay a certificate authority (CA) that other computers trust to sign your certificate for you. Certificate authorities are supposed to have integrity, so they are supposed to actually make sure that when they sign certificates, the person they give the certificate to actually owns the domain.

3. You install your signed certificate and use it to prove that you are really you.
