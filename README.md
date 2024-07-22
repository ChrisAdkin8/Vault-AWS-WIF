# Overview

This repo is intended to help demonstrate workdload identity federation (WIF) which is available with HashiCorp Vault Enterprise from version 1.16 onwards.


# Instructions

## Configuring OIDC and JWKS Endpoint Access to AWS

1. The demo in this repo has been tested on MacOs, the first step is to install ngrok:
```
brew install ngrok
```

2. Connect your ngrok agent to your ngrok account (sign up for an ngrok account if you haven't done so already) and then copy your ngrok authtoken from your ngrok dashboard:
```
ngrok config add-authtoken <TOKEN>
```   

3. Obtain the name of a hosted domain from the Route 53 section of the AWS console:
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/01-route-53-hosted-domain.png?raw=true">

4. Create a new domain in the ngrok enterprise console based on the hosted domain obtained from the previous step, append the name of the hosted domain onto 'vault.', copy the 
cname value for use in the next step:
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/02-ngrok-domain-creation.png?raw=true">

5. Create a CNAME record for the hosted domain, use vault as the sub domain and the cname value obtained in the previous step, hit "Create records":
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/03-route-53-creation.png?raw=true">

6. Return to the ngrok enterprise page and note the message stating a TLS certificate is being created, the amount of time you will have to wait will be dependent upon
the TLS timeout specified when the cname record was created, i.e. the longer this is, the longer you will have to wait for a TLS cert:  
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/04-ngrok-cert-creation.png?raw=true">

7. Clicking on the screen icon containing >_ in the top right hand corner of the ngrok page will provide the ngrok command for creating a tunnel, this will be required later:
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/05-ngrok-start-a-tunnel.png?raw=true">

8. Once your domain is ready - a TLS cert will appear on the main domain page to the right of your domain name:
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/06-ngrok-domain-ready.png?raw=true">

9. Create a tunnel as follows from within zsh:
```
ngrok http http://localhost:8200  --domain=<YOUR_DOMAIN_NAME>
```

## Spinning Up Vault on Docker

Follow [these instructions](https://github.com/ChrisAdkin8/Docker-Vault) for spinning up Vault inside a docker container, note that you will require:
- Docker to be installed on MacOS
- A Vault Enterprise license, specifically you will require the license key for this

## HashiCorp Vault WIF Tutorial

1. Confirm the VAULT_ADDR is set to http://localhost:8200
```
% env | grep VAULT_ADDR
```

2. Check the status of Vault:
```
% vault status 
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.16.2+ent
Build Date      2024-04-22T16:26:15Z
Storage Type    file
Cluster Name    vault-cluster-de2d1755
Cluster ID      8f8dd09e-2854-e91a-1988-950db4ad64b9
HA Enabled      false
```

3. Open the OIDC endpoint in a browser in order to obtain the JWKS URI, for example if the domain you are using is:
```
vault.chris-adkin.sbx.hashidemos.io
```
the URL you need to obtain in a browser is:

https://vault.chris-adkin.sbx.hashidemos.io/v1/identity/oidc/plugins/.well-known/openid-configuration

this should produce output (pretty printed) which looks similar to this:
```
{
  "issuer": "http://127.0.0.1:8200/v1/identity/oidc/plugins",
  "jwks_uri": "http://127.0.0.1:8200/v1/identity/oidc/plugins/.well-known/keys",
  "response_types_supported": [
    "id_token"
  ],
  "subject_types_supported": [
    "public"
  ],
  "id_token_signing_alg_values_supported": [
    "RS256",
    "RS384",
    "RS512",
    "ES256",
    "ES384",
    "ES512",
    "EdDSA"
  ]
}
```

4. To obtain the certificate for the fingerprint, use the following curl command:
```
openssl s_client -servername <YOUR_DOMAIN_NAME> -showcerts -connect <YOUR_DOMAIN_NAME>:443
```

if your domain name is (for example) vault.chris-adkin.sbx.hashidemos.io, use:
```
openssl s_client -servername vault.chris-adkin.sbx.hashidemos.io -showcerts -connect vault.chris-adkin.sbx.hashidemos.io:443
```

scroll up the output from this command and grab the latest certificate block that appears in it, for example:
```
.
.
.
-----BEGIN CERTIFICATE-----
MIIEVzCCAj+gAwIBAgIRAIOPbGPOsTmMYgZigxXJ/d4wDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMjQwMzEzMDAwMDAw
WhcNMjcwMzEyMjM1OTU5WjAyMQswCQYDVQQGEwJVUzEWMBQGA1UEChMNTGV0J3Mg
RW5jcnlwdDELMAkGA1UEAxMCRTUwdjAQBgcqhkjOPQIBBgUrgQQAIgNiAAQNCzqK
a2GOtu/cX1jnxkJFVKtj9mZhSAouWXW0gQI3ULc/FnncmOyhKJdyIBwsz9V8UiBO
VHhbhBRrwJCuhezAUUE8Wod/Bk3U/mDR+mwt4X2VEIiiCFQPmRpM5uoKrNijgfgw
gfUwDgYDVR0PAQH/BAQDAgGGMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
ATASBgNVHRMBAf8ECDAGAQH/AgEAMB0GA1UdDgQWBBSfK1/PPCFPnQS37SssxMZw
i9LXDTAfBgNVHSMEGDAWgBR5tFnme7bl5AFzgAiIyBpY9umbbjAyBggrBgEFBQcB
AQQmMCQwIgYIKwYBBQUHMAKGFmh0dHA6Ly94MS5pLmxlbmNyLm9yZy8wEwYDVR0g
BAwwCjAIBgZngQwBAgEwJwYDVR0fBCAwHjAcoBqgGIYWaHR0cDovL3gxLmMubGVu
Y3Iub3JnLzANBgkqhkiG9w0BAQsFAAOCAgEAH3KdNEVCQdqk0LKyuNImTKdRJY1C
2uw2SJajuhqkyGPY8C+zzsufZ+mgnhnq1A2KVQOSykOEnUbx1cy637rBAihx97r+
bcwbZM6sTDIaEriR/PLk6LKs9Be0uoVxgOKDcpG9svD33J+G9Lcfv1K9luDmSTgG
6XNFIN5vfI5gs/lMPyojEMdIzK9blcl2/1vKxO8WGCcjvsQ1nJ/Pwt8LQZBfOFyV
XP8ubAp/au3dc4EKWG9MO5zcx1qT9+NXRGdVWxGvmBFRAajciMfXME1ZuGmk3/GO
koAM7ZkjZmleyokP1LGzmfJcUd9s7eeu1/9/eg5XlXd/55GtYjAM+C4DG5i7eaNq
cm2F+yxYIPt6cbbtYVNJCGfHWqHEQ4FYStUyFnv8sjyqU8ypgZaNJ9aVcWSICLOI
E1/Qv/7oKsnZCWJ926wU6RqG1OYPGOi1zuABhLw61cuPVDT28nQS/e6z95cJXq0e
K1BcaJ6fJZsmbjRgD5p3mvEf5vdQM7MCEvU0tHbsx2I5mHHJoABHb8KVBgWp/lcX
GWiWaeOyB7RP+OfDtvi2OsapxXiV7vNVs7fMlrRjY1joKaqmmycnBvAq14AEbtyL
sVfOS66B8apkeFX2NY4XPEYV4ZSCe8VHPrdrERk2wILG3T/EGmSIkCYVUMSnjmJd
VQD9F6Na/+zmXCc=
-----END CERTIFICATE-----
---
Server certificate
subject=CN=vault.chris-adkin.sbx.hashidemos.io
issuer=C=US, O=Let's Encrypt, CN=E5
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 2416 bytes and written 407 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_128_GCM_SHA256
Server public key is 256 bit
This TLS version forbids renegotiation.
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_128_GCM_SHA256
    Session-ID: 3B68B23120C80C29E666642990AF42BD3C53E0BD2312F1FB8C1E74F27ED3D6B4
    Session-ID-ctx: 
    Resumption PSK: B1E781631C09AA3104895178579954C9C322821260F72AA2FDE092CA1BE6DA0A
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 604800 (seconds)
    TLS session ticket:
    0000 - 90 31 fe 03 75 39 43 25-1d ab 67 8d fe 8d 53 2d   .1..u9C%..g...S-
    0010 - ae 43 29 df 19 6d c3 21-a9 8a 7a c1 c8 5d 19 b2   .C)..m.!..z..]..
    0020 - 00 84 06 f0 5e fa 15 15-ea fd c5 27 78 df 7c 66   ....^......'x.|f
    0030 - 53 f6 54 6d d3 30 34 26-04 88 1c d3 a3 c3 5a 28   S.Tm.04&......Z(
    0040 - c1 ca 60 04 28 e2 6c 4a-5c b8 56 87 37 3b 44 61   ..`.(.lJ\.V.7;Da
    0050 - 1e 7a 97 7c 47 b5 e1 17-df f3 6c 94 c2 3b 16 ac   .z.|G.....l..;..
    0060 - 7f 68 b5 5a 90 6c 15 35-93                        .h.Z.l.5.

    Start Time: 1721662261
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
```

Save the certificate a file called certificate.crt.

5. Execute the following command in order to obtain the thumb print:
```
openssl x509 -in certificate.crt -fingerprint -sha1 -noout | awk '{ print substr($2, 13, 59) }' | awk '{gsub(":", ""); print}'
```


  



