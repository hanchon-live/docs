---
title: "Simple guide to install and configure NGINX on Ubuntu"
date: 2021-04-26T22:10:30+01:00
categories:
  - guide
tags:
  - nginx
  - ubuntu
  - devops
  - https
  - certbot
  - debian
last_modified_at: 2021-04-27T22:10:30+01:00
---
In this post we are going to install and configure [nginx](https://www.nginx.com/) on an [Ubuntu](https://ubuntu.com/) or any [Debian](https://www.debian.org/) server. We are also going to use [certbot](https://certbot.eff.org/) to set the HTTPs certificate (Let's encrypt) to host our webpage only using `https`.

{% include toc icon="cog" title="Content" %}
{% include figure image_path="/assets/posts/nginx/header.png" alt="NGINX ubuntu certbot and let's encrypt - keybindings" caption="" %}
# NGINX
## Installation:
We can use `apt` to install the web server.

NOTE: make sure you don't have anything running on the port `80`, to avoid restarting the server after the installation.

``` sh
sudo apt update
sudo apt install nginx
```

If you have a firewall, you should set your rules after this step.

## Check the server status:
To check the status we can just run:
```sh
systemctl status nginx
```

It should return something like this:
```sh
$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2021-04-26 22:14:56 UTC; 1min 18s ago
     Docs: man:nginx(8)
  Process: 28780 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS
  Process: 28781 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 28782 (nginx)
    Tasks: 3 (limit: 4665)
   Memory: 3.6M
   CGroup: /system.slice/nginx.service
           ├─28782 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─28783 nginx: worker process
           └─28784 nginx: worker process

Apr 26 22:14:56 burninstone-new systemd[1]: Starting A high performance web server and a reverse proxy server...
Apr 26 22:14:56 burninstone-new systemd[1]: Started A high performance web server and a reverse proxy server.
```
If the server is **active** (running), everything is correct. You can check it in any browser using the url `http://your-ip`


## Useful commands:
- **Stop** the web server:
``` sh
sudo systemctl stop nginx
```

- **Start** the web server:
``` sh
sudo systemctl start nginx
```

- **Restart** the web server:
``` sh
sudo systemctl restart nginx
```

- **Check** if your configuration file syntax is valid:
```sh
sudo nginx -t
```

- **Reload** the web server (after making changes on your config file):
``` sh
sudo systemctl reload nginx
```

## Create an index.html:
We are going to configure our server to host an index file.

This guide is going to use `your_domain` as an example, but you should replace that variable with your own domain.

- Create the folder:
``` sh
sudo mkdir -p /var/www/your_domain/html
```

- Set the owner:
```sh
sudo chown -R $USER:$USER /var/www/your_domain/html
```

- Set the permissions:
```sh
sudo chmod -R 755 /var/www/your_domain
```

- Create the index file:
```sh 
sudo vim /var/www/your_domain/html/index.html
```
  - NOTE-1: I'm using [VIM](https://www.vim.org/) to edit my files, but you can use any other editor, for example, [NANO](https://www.nano-editor.org/).
  - NOTE-2: To install `vim` on `Ubuntu`: `sudo apt install vim`.
  - NOTE-3: To exit `vim`, save and quit pressing: `esc` + `:wq` + `enter`.

- Code the index.html:
  - NOTE: to paste on `vim` press `insert`.

``` html
<html>
    <head>
        <title>Hanchon test</title>
    </head>
    <body>
        <h1>Testing NGINX on Ubuntu</h1>
    </body>
</html>
```

## Configure your domain:
NOTE: In every step change the `your_domain` value.

Let's start creating a new file with the configuration for our domain.

``` sh
sudo vim /etc/nginx/sites-available/your_domain
```

After creating the domain, let's serve a webpage with this configuration:


```
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
```


Create a symbolic link to the `sites-enabled` folder, so `NGINX` knows that we want to use this configuration.
```sh
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

Check the configuration syntax and if everything is ok, restart `NGINX`:
```sh
sudo nginx -t
sudo systemctl restart nginx
```

NOTE: if you are using Angular builds, add this line to avoid having errors when refreshing the page.
```
location / {
    root   /var/www/ethics_demo/html;
    try_files $uri $uri/ /index.html;
    index  index.html;
}

```

## Use NGINX as a proxy:
We can configure our webserver to redirect the request to another endpoint, for example an application running locally in our server:
```
server {
    listen 80;
    listen [::]:80;
    server_name your_domain www.your_domain;
    location /api/ {
        proxy_pass http://127.0.0.1:7000/;
        include proxy_params;
    }
}
```

NOTE: If you are using `FastAPI` as your API, like it was explained in previous guides, you may must to add your `root_path` to the `FastAPI` constructor: `app = FastAPI(root_path='/api')`.

# Certificates
We are going to use `certbot` to create, install and renew free certificates (**Let's Encrypt**).

## Install Certbot:
We are going to install `certbot` using `snap`:
- Let's install snap if needed:
``` sh
sudo apt install snapd;
```
- Let's install `core`:
``` sh
sudo snap install core;
```
  - Note: if you are having problems, you should close the `terminal` and reopen it, so the `snap` paths are added to your `terminal`.

- Install `certbot`:
``` sh
sudo snap install --classic certbot;
```

- Make a link to `/usr/bin` to use it:
``` sh
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
## Certbot usage:
- Create and install the certificate:
``` sh
sudo certbot --nginx -d your_domain
```
  - NOTE: if you want to create certificates for all your domains, you can ignore the `-d` param.

- Auto-update certificates:
  - Certbot already updates your certificates before they expire.
  - You can test the renew process using this command `sudo certbot renew --dry-run`

## Test your webpage:
The last step is to test if everything is working as intended:
- Enter to **https://your_domain** and it should work.
- Enter to **http://your_domain** and you should be redirected to **https://your_domain**.