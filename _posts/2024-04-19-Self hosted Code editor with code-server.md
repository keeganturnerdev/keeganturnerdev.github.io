---
title: Self hosted Code editor with code-server
date: 2024-04-19
categories: [cloud]
tags: [cloud]
image: /assets/codeserver.jpg
---

<br>

**Host your own code editor on your server which can be accessed remotely from any device**

<br>

***Self-hosting your code editor is like having your own playground for coding. You get to tweak and customize it however you want, making it the perfect fit for your coding style. Plus, it keeps your code and data safe and sound on your own turf, away from prying eyes. By self-hosting, you can also speed up your workflow since you're not relying on distant servers. Best of all, it's your setup, so you're not tied down by subscriptions or someone else's rules—total coding freedom!***

<br>

Step 1 - System Updates 

SSH into your server or cloud instance and perform system updates

```
sudo apt update -y
```

Apply the available updates

```
sudo apt dist-upgrade -y
```
<br>

Step 2 - Download code-server
<br>

Download the installation package for your operating system 
from the <a href="https://github.com/coder/code-server/releases" target="_blank">code-server Github repository </a> 

The following commands apply to debian based distributions:
<br>

```
sudo rpm -i code-server-$VERSION-amd64.rpm
```

```
sudo systemctl enable --now code-server@$USER
```
Replace '$USER' with your server username

<br>

Navigate to .config/code-server and change the bind address in the 'config.yaml' file to your server IP address.
Ensure that port 8080 is allowed on your firewall.
<br>

Access code-server on your-ip-address:8080 
The password can be found in .config/code-server/config.yaml.


