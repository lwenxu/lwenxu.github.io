---
title: Mac配置sshd
date: 2017-05-13 06:53:23
tags:
categories:
  -Mac
---
修改sshd_config文件把password验证方式打开
修改这个文件要用sudo

>PasswordAuthentication yes

```bash
sudo ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
sudo ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
sudo  ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
sudo  ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
sudo /usr/sbin/sshd
```
