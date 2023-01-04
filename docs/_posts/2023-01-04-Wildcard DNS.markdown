---
layout: post
title:  "Wildcard DNS on local services"
date:   2023-01-04 11:00:10 +0000
categories: DNS homelab
---

# Introduction
Purpose is to enable using proper certificates inside the homelab. This is solved by the following steps
* registering a wildcard A-record at your DNS-provider (e.g. cloudflare)
* proving you are the owner with the help of certbot 
* configuring local dns for the local wildcard record, pointing to a reverse proxy
* setting up a reverse proxy e.g. traefik or NGNX with rules to route each hostname to the correct service

# Register wildcard DNS at DNS-provider
If using cloudflare, see: https://developers.cloudflare.com/dns/manage-dns-records/reference/wildcard-dns-records/

register an A-record for something like local.MYDOMAIN.COM or l.MYDOMAIN.COM, pointing to your fixed IP (if having dynamic IP, set that up)

# Prove you are the owner or the domain using certbot
## Install certbot on a local server, not exposed
This is the recommended, more safe(?) alternative.
On one server on the local network, install certbot
Assuming DNS-provider is cloudflare, follow: https://labzilla.io/blog/cloudflare-certbot

In short:
### 1) Install certbot

Follow: https://certbot.eff.org/instructions?ws=other&os=ubuntufocal
```console
sudo snap install core; sudo snap refresh core

sudo apt-get install python3-certbot-dns-cloudflare
sudo mkdir /root/.secrets/
sudo touch /root/.secrets/cloudflare.ini
```
Note1: Needs to be installed on VM. Does not work on Proxmox LXC due to a mount problem with snapd for the moment (seems to be a workaround but it has some drawbacks, see https://forum.proxmox.com/threads/ubuntu-snaps-inside-lxc-container-on-proxmox.36463/#post-230060)

Note2: There are multiple tutorials out there which are not up to date. Make sure to use the latest from certbot

### 2) Get API key from cloudflare
API-key is created from cloudflare dashboard
* https://dash.cloudflare.com/
* Go to my profile --> API Tokens
* Create token with permissions to edit zone DNS


### 3) Configure file for certbot with your credentials
```console
sudo vi /root/.secrets/cloudflare.ini
```
update file with your details
```yaml
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = YOUR-TOKEN
```
### 4) Secure secrets files
```console
sudo chmod 0700 /root/.secrets/
sudo chmod 0400 /root/.secrets/cloudflare.ini
```

### 5) Request certificate
* Request "certificate only"

```console
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d example.com,*.example.com --preferred-challenges dns-01
 ```

### 6) Use the retrieved certificates
If successfully received certificate.
* Certificate is saved at: /etc/letsencrypt/live/zalfnet.com/fullchain.pem
* Key is saved at: /etc/letsencrypt/live/zalfnet.com/privkey.pem
* Note:
* These files will be updated when the certificate renews.
* Certbot has set up a scheduled task to automatically renew this certificate in the background.

## Notes
The expiry date for the API-key is set when creating it in cloudflare. Take care when it expires. Also probably good to not make a too long expiry time if it is leaked

## Alternative: Set up server with port 80 exposed 
Using a server in you DMZ which is isolated from the rest of the network, install certbot: https://certbot.eff.org/ and e.g. https://certbot.eff.org/instructions?ws=other&os=ubuntufocal
This require that you open up and forward port 80 to the server running certbot. Works well when using other DNS providers than Cloudflare which does not have the API-key method