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
## Set up server
Using a server in you DMZ which is isolated from the rest of the network, install certbot: https://certbot.eff.org/ and e.g. https://certbot.eff.org/instructions?ws=other&os=ubuntufocal

## Request certificate
* Setup port forwarding of port 80 to the "Certbot server"
* Request "certificate only"

```console
sudo certbot certonly -d *.l.zalfnet.com --manual --server https://acme-v02.api.letsencrypt.org/directory
 ```

