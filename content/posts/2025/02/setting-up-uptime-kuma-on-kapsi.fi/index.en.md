---
translationKey: setting-up-uptime-kuma-on-kapsi.fi
title: Setting up Uptime Kuma on Kapsi.fi
draft: false
langkey: ''
slug: ''
date: 2025-02-18
categories:
  - tutorial
tags:
  - tech
  - selfhosting
description: asd
---

Small tutorial on how to run [Uptime Kuma](https://github.com/louislam/uptime-kuma) on Kapsi.fi shared hosting environment without Docker.

Update: 2026-07-02 - Migration to v2 Uptime Kuma and webapp-trixie on Kapsi

## Prerequisities:

- Kapsi account
- Port opened to webapp servers. Request [this](https://www.kapsi.fi/palvelut/portit.html) from Kapsi admins.
- ssh
- (Optional) Own domain address
<!--more-->

## Steps:

1. Ssh to the webapp-trixie server `ssh <account>@webapp-trixie.kapsi.fi`
2. Install `nvm` to install Nodejs. See the recent installation guide from [here](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating).\
I just used the wget snippet and then added below to `.bash_profile`

```plain
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

After this run `nvm install` which at the writing install NodeJS v23.  
You can also run `nvm install --lts` to install latest LTS version.

3. Change directory to desired domain you want to install eg `cd /sites/domain.com/www/`
4. Create a `.htaccess` file with following content.\
 This redirects requests from `domain.com` to the underlying web server running in `webapp-bullseye.n.kapsi.fi:<PORT>`

```plain
# Uudelleenohjaus http -> https
RewriteEngine On
RewriteCond %{ENV:HTTPS} !on
RewriteRule (.*) https://%{SERVER_NAME}%{REQUEST_URI} [R=301,L]

RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteCond %{HTTP:Connection} upgrade [NC]
RewriteCond %{REQUEST_URI} ^/(.*)$
RewriteRule ^(.*) ws://webapp-trixie.n.kapsi.fi:<PORT>/$1  [P]
RewriteRule ^(.*)$ http://webapp-trixie.n.kapsi.fi:<PORT>/$1 [P]
```

6. Follow the official instructions for non-Docker installation [here](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install#-non-docker).

- `git clone https://github.com/louislam/uptime-kuma.git .`
- Create `.env` file to the root installation with this content

```plain
 UPTIME_KUMA_HOST= webapp-bullseye.n.kapsi.fi
  UPTIME_KUMA_PORT=<PORT>"
```

- Do a test run with `node server/server.js`, and access your domain eg `domain.com`

7. Install `pm2` for running as a background processes

- `npm install pm2 -g && pm2 install pm2-logrotate`
- Start it `pm2 start server/server.js --name uptime-kuma`

6. Create cronjob entry for starting the server on reboot (Doesn't work for webapp-trixie, use systemd)

- `crontab -e`
- Add following `@reboot cd ~/sites/domain.com/www && pm2 start server/server.js --name uptime-kuma`


## v2 migration

Easy to do, just follow the general steps from official instruction [here](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%86%99-How-to-Update#--non-docker). 

Then start up the service and check logs with `pm2 logs uptime-kuma` for migration process.

Also make a systemd user service instead of crontab

```
~/.config/systemd/user/uptimekumaservice

[Unit]
Description=Uptime Kuma startup

[Service]
WorkingDirectory=~/sites/domain.com/www
ExecStart=pm2 start server/server.js --name uptime-kuma
Restart=always

[Install]
WantedBy=default.target
```
