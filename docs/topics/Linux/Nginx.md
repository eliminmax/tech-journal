<!--
SPDX-FileCopyrightText: 2022 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# nginx

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
