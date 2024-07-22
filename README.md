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

vault.chris-adkin.sbx.hashidemos.io

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



  



