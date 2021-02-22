---
layout: post
title:  "[EN] Let's Encrypt on myNode for BTCPayServer"
description: How to get Let\'s Encrypt working on myNode for HTTPS
tags: lightning-network lets-encrypt mynode raspberrypi btcpayserver
---

## Prerequisites
Because almost all ports are taken on myNode, we need to choose some, which is not. By random.org I've choosed 8443. So what you need to do is following:
 - port forward on your router 80 -> ip_mynode:8443 (for Let's encrypt)
 - port forward on your router 443 -> ip_mynode:49393 (for BTCPayServer)

## Login to myNode

You can use [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), login is `admin`, password is same as you are using to login to administration of myNode. Then run following command to get a root (if it ask for password, it's the same as for user admin):
```
sudo su -
```

## Backup of certificate from myNode

```
cp /home/bitcoin/.mynode/https/myNode.local.crt /home/bitcoin/.mynode/https/myNode.local.crt.bak

cp /home/bitcoin/.mynode/https/myNode.local.key /home/bitcoin/.mynode/https/myNode.local.key.bak
```

## Installation of acme.sh
You can install acme.sh (it's a script for Let's Encrypt) - [acme.sh github page](https://github.com/acmesh-official/acme.sh) (if someone would like to read the source code).

```
curl https://get.acme.sh | sh
```

## Certificate generation
In following set of commands, change DOMAIN variable to domain which you own and want. After these commands acme.sh is going to renew certificate every 3 months (It's important to keep port 8443 opened on your router!).

```
DOMAIN="pay.domain.com"

acme.sh --issue -d ${DOMAIN} --standalone --httpport 8443

acme.sh -d ${DOMAIN} --install-cert --fullchain-file /home/bitcoin/.mynode/https/myNode.local.crt \
--key-file /home/bitcoin/.mynode/https/myNode.local.key \
--reloadcmd "systemctl restart nginx.service"
```

## Rollback
In case something horrible happen and you want to rollback:

```
cp /home/bitcoin/.mynode/https/myNode.local.crt.bak /home/bitcoin/.mynode/https/myNode.local.crt

cp /home/bitcoin/.mynode/https/myNode.local.key.bak /home/bitcoin/.mynode/https/myNode.local.key

systemctl restart nginx.service
```