---
title: Uptime Kuman käyttöönotto Kapsi.fi-sivustolla
draft: false
date: 2025-02-18
description: asd
categories:
  - tutorial
tags:
  - tech
  - selfhosting
slug: uptime-kuma-kayttoonotto-kapsi-fi
---
Pieni tutoriaali [Uptime Kuman](https://github.com/louislam/uptime-kuma) käyttämisestä Kapsi.fi- ympäristössä ilman Dockeria.

## Vaatimukset:

-   Kapsi-tili
-   Portti avattu web-sovelluspalvelimille. Pyydä [tätä](https://www.kapsi.fi/palvelut/portit.html) Kapsin ylläpitäjiltä.
-   ssh
-   (Valinnainen) Oma verkkotunnusosoite
<!--more-->
## Vaiheet:

1.  Ssh:ta webapp-bullseye-palvelimelle `ssh <account>@webapp-bullseye.kapsi.fi`
2.  Asenna `nvm` asentaaksesi Nodejs:n. Katso uusin asennusopas [täältä](https://github.com/nvm-sh/nvm?tab=readme-ov-file#installing-and-updating) .  
    Käytin itse  wget- rimpuaja lisäsin sitten alla olevan `.bash_profile` tiedostoon

```plain
`&#32;export NVM_DIR="$HOME/.nvm"`
`[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm`
`[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion`T
```

Tämän jälkeen aja `nvm install` , joka  asentaa NodeJS v23:n.  
Voit myös ajaa `nvm install --lts` asentaaksesi uusimman LTS-version.

3.  Vaihda hakemistoon, johon haluat asentaa, esim. `cd /sites/domain.com/www/`
    
4.  Luo `.htaccess` -tiedosto seuraavalla sisällöllä.  
    Tämä ohjaa pyynnöt osoitteesta `domain.com` `webapp-bullseye.n.kapsi.fi:<PORT>`
    

```plain
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

5.  Seuraa Uptime Kuman non-docker ohjeita [täällä](https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install#-non-docker) .

-   `git clone https://github.com/louislam/uptime-kuma.git .`
-   Luo `.env` -tiedosto pääasennukseen tällä sisällöllä

```plain
 UPTIME_KUMA_HOST= webapp-bullseye.n.kapsi.fi UPTIME_KUMA_PORT=<PORT>"
```

-   `node server/server.js` ja tarkista toimivuus menemällä omaan domainiin esim `domain.com`

6.  Asenna `pm2` , jotta serveri toimii taustalla

-   `npm install pm2 -g && pm2 install pm2-logrotate`
-   Käynnistä se `pm2 start server/server.js --name uptime-kuma`

7.  Luo cronjob palvelimen käynnistämiseksi uudelleenkäynnistyksen yhteydessä

-   `crontab -e`
-   Lisää seuraava komento `@reboot cd ~/sites/domain.com/www && pm2 start server/server.js --name uptime-kuma`

Siinä se!
