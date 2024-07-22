# Overview

This repo is intended to help demonstrate workdload identity federation (WIF) which is available with HashiCorp Vault Enterprise from version 1.16 onwards.

# Instructions

## Configuring OIDS and JWKS Endpoint Access to AWS

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

5. Create a CNAME record for the hosted domain, use vault as the sub domain and the cname value obtained in the previous step:
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/03-route-53-creation.png?raw=true">
4444
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/04-ngrok-cert-creation.png?raw=true">
5555
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/05-ngrok-start-a-tunnel.png?raw=true">
6666
<img style="float: left; margin: 0px 15px 15px 0px;" src="https://github.com/ChrisAdkin8/Vault-AWS-WIF/blob/main/png_images/06-ngrok-domain-ready.png?raw=true">



