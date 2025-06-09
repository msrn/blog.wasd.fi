---
date: '2025-05-21'
draft: false
title: 'Mounting SMB shares (and others) on Linux'
categories:
 - tutorial
tags: 
 - tech
 - linux
---



```
[Unit]
Description=Mount <name> SMB
After=network-online.target
Wants=network-online.target

[Mount]
Type=rclone
What=<name>:/
Where=/var/home/<user>/mnt/<name>/
Options=config=/var/home/<user>/.config/rclone/rclone.conf,vfs-cache-mode=full

[Install]
WantedBy=default.target
```