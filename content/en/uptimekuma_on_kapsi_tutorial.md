---
date: '2025-02-18'
draft: false
title: 'Setting up Uptime Kuma on Kapsi.fi'
categories:
 - tutorial
tags: 
 - tech, selfhosting
---

Small tutorial on how to run [Uptime Kuma](https://github.com/louislam/uptime-kuma) on Kapsi.fi shared hosting environment without Docker.

Prerequisities:
 - Kapsi account
 - Port opened to webapp servers. Request [this](https://www.kapsi.fi/palvelut/portit.html) from Kapsi admins.
 - ssh
 - (Optional) Own domain address

Steps:

1. Ssh to the webapp-bullseye server `ssh <account>@webapp-bullseye.kapsi.fi`

2. Install `nvm` to install Nodejs. See the recent installation guide from [here](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating) 
    I just used the wget snippet and then added below to `.bash_profile`
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

After this run `nvm install` which at the writing install NodeJS v23. You can also run `nvm install --lts` to install latest LTS version.

3. Change directory to desired domain you want to install eg `cd /sites/domain.com/www/`
   
5. Create a `.htaccess` file with following content. This redirects requests from `domain.com` to the underlying web server running in `webapp-bullseye.n.kapsi.fi:<PORT>`
```
# Uudelleenohjaus http -> https
RewriteEngine On
RewriteCond %{ENV:HTTPS} !on
RewriteRule (.*) https://%{SERVER_NAME}%{REQUEST_URI} [R=301,L]

RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteCond %{HTTP:Connection} upgrade [NC]
RewriteCond %{REQUEST_URI} ^/(.*)$
RewriteRule ^(.*) ws://webapp-bullseye.n.kapsi.fi:<PORT>/$1  [P]
RewriteRule ^(.*)$ http://webapp-bullseye.n.kapsi.fi:<PORT>/$1 [P]
```
4. Follow the official instructions for non-Docker installation [here](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install#-non-docker).
  - `git clone https://github.com/louislam/uptime-kuma.git .`
  - Create `.env` file to the root installation with this content
  ```
  UPTIME_KUMA_HOST= webapp-bullseye.n.kapsi.fi
  UPTIME_KUMA_PORT=<PORT>"
  ```
  - Do a test run with `node server/server.js`. And accessing your domain eg `domain.com`

5. Install `pm2` for running as a background processes
  - `npm install pm2 -g && pm2 install pm2-logrotate`
  - Start it `pm2 start server/server.js --name uptime-kuma`
6. Create cronjon entry for starting the server on reboot
- `crontab -e`
- Add following `@reboot cd ~/sites/domain.com/www && pm2 start server/server.js --name uptime-kuma
