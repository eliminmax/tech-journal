<!--
SPDX-FileCopyrightText: 2022 - 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Linux: nginx

<!-- vim-markdown-toc GitLab -->

* [Install nginx](#install-nginx)
  * [Debian family](#debian-family)
  * [RHEL 8 family](#rhel-8-family)
* [Configure nginx to use HTTPS](#configure-nginx-to-use-https)

<!-- vim-markdown-toc -->

## Install nginx

Installing nginx is simple on most modern distros - install the nginx package with the default
package manager.

### Debian family

`apt install nginx` (as root/with `sudo`)

### RHEL 8 family

`dnf install nginx` (as root/with `sudo`)

## Configure nginx to use HTTPS

Once nginx is installed, and [you have a signed certificate](../Networking/Security/TLS-Certificate-Signing.md), navigate to /etc/nginx/sites-available, then edit the configuration files to use TLS/SSL - the following example redirects all HTTP requests to HTTPS:

```nginx
server {
        listen 80 default_server;
        index index.html;
        server_name _;
        # redirect to HTTPS version of site
        return 301 https://for.example$request_uri;
}
server {
        root /var/www/html;
        listen 443 ssl;
        ssl_certificate /etc/ssl/for.example.crt;
        ssl_certificate_key /etc/ssl/for.example.key;
        server_name for.example;
        location / {
                try_files $uri $uri/ =404;
        }
}
```
