# Linux: nginx

> <details>
> <summary>Meaning of different command prompts</summary>
> Unix/Linux: <code>$</code>: can be run as normal user<br>
> Unix/Linux: <code>#</code>: must be run as root (or with <code>sudo</code>)<br>
> Windows: <code>></code>: Command Prompt or PowerShell<br>
> Windows: <code>PS></code>: PowerShell only<br>
> Unix/Linux and Windows: <code>$/></code>,<code>#/></code>: Works in Windows and Unix/Linux.
> </details>

## Install nginx

Installing nginx is simple on most modern distros - install the nginx package with the default
package manager.

### Debian family

`# apt install nginx`

### RHEL 8 family

`# dnf install nginx`

## Configure nginx to use HTTPS

Once nginx is installed, and [you have a signed certificate](/topics/Networking/Security/TLS-Certificate-Signing), navigate to /etc/nginx/sites-available, then edit the configuration files to use TLS/SSL - the following example redirects all HTTP requests to HTTPS:

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
