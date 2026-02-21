---
title: Mount SMB shares on Linux
draft: false
slug: ''
date: 2026-02-21
categories:
  - tech
tags:
  - tutorial
---

I have been using Linux distribution called Aurora Linux [^1] well over year as my main distro. I did pretty much cold turkey switch from Windows 10, as the Window s boot drive's bootloader just failed for some reason.

I also have my own server running, and I like to use SMB as I can't then use Unraid's Recycle Bin plugin functionalities in case of accidentally deleted files.

Anyway here are two methods to mount SMB shares on immutable distro's [^2]

## fstab

1. Normal mount
`sudo mount -t cifs -o username=<user>,password=<password>,vers=2.0 //localhost/Media /mnt/smb/Media`

2. Add to fstab
```
//localhost/Media    /mnt/smb/Media   cifs  auto,x-gvfs-name=Media,username=<user>password=<password>,uid=1000,gid=100,file_mode=0664,dir_mode=0775,vers=2.0    0    0
```

## Rclone

This is my preferred way, as I can also use this mount Fastmail's Webdav share [^3].

1. Create rclone config as such

```
[Server]
type = smb
host = localhost
user = <username>
pass = <passwordbase64>
```
2. Create systemd mount unit, and name as `var-home-user-mnt-smb.mount`

Save this to `/home/user/.config/systemd/user/

Note that the mount unit's filename has to be the same one as the target where the SMB share is mounted.

```
[Unit]
Description=Mount smb
After=network-online.target
Wants=network-online.target

[Mount]
Type=rclone
What=Server:/
Where=/var/home/user/mnt/server/
Options=config=/var/home/user/.config/rclone/rclone.conf,vfs-cache-mode=full
TimeoutSec=120

[Install]
WantedBy=default.target
```
3. Then do some systemctl commands

```
sudo systemctl daemon-reload
sudo systemctl enable var-home-user-mnt-smb.mount
sudo systemctl start var-home-user-mnt-smb.mount

```
[^1]: https://getaurora.dev
[^2]: https://forum.manjaro.org/t/root-tip-how-to-systemd-mount-unit-samples/1191
[^3]: https://sleeplessbeastie.eu/2017/09/25/how-to-mount-webdav-share-using-systemd/
