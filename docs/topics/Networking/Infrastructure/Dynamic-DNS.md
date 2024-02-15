# Networking Infrastructure: Dynamic DNS

<!-- vim-markdown-toc GitLab -->

* [DuckDNS](#duckdns)
* [YDNS (NOT RECOMMENDED)](#ydns-not-recommended)
    * [Linux](#linux)
    * [Windows](#windows)

<!-- vim-markdown-toc -->

Dynamic DNS (a.k.a. DDNS) services automatically update DNS records whenever a device's public IP changes, so that there is a consistent way to access a device which might not have a persistent public IP. There are various DDNS providers out there, both free and paid. When setting up DDNS on my RHEL EC2 instance, I found the free DDNS provider YDNS.io to be quite easy to use, but I've since switched to DuckDNS, because it sets all subdomains of the domain automatically.

**Note:** I've noticed when testing this that it can take a few minutes for the update to the DNS record to propogate.

## DuckDNS

> DuckDNS is a great service, in my experience. It does the same thing as YDNS, but it also automatically sets all subdomains of each of your subdomains (subsubdomains?) to the same ip as the subdomain.

0. Log in to [DuckDNS](https://www.duckdns.org) using a Google, Twitter, or GitHub account, then add a domain.

1. Go to the [DuckDNS install instructions](https://www.duckdns.org/install.jsp), and select an install method, and follow the instructions.

## YDNS (NOT RECOMMENDED)

> While I originally recommended this method, I don't anymore. YDNS and DuckDNS both limit you to five subdomains, DuckDNS automatically sets all subdomains of that subdomain as well.

0. Create an account on [YDNS.io](https://ydns.io)

1. Launch the EC2 instance you want to configure

2. On your YDNS account home, click the ***+ New Host*** button, choose a domain from the dropdown menu, and enter a name, then enter the EC2 instance public IP Address in the content field.

#### Linux

This section assumes you are logged in as root. If not, switch to root with `sudo -i`, `sudo -s`, or equivalent.

3. Install cURL, if necessary, then run the following: `curl -o /root/ddns-updater.sh https://raw.githubusercontent.com/ydns/bash-updater/master/updater.sh`

5. Set the permissions to 700 for **ddns-updater.sh**: `chmod 700 ddns-updater.sh`

6. Edit *ddns-updater</nolink>.sh*, so that the values of `$YDNS_USER` and `$YDNS_PASSWD` reflect either your login or [api credentials](https://ydns.io/user/api), and the change the `$YDNS_HOST` value to reflect the name and domain you chose in Step 2

7. Edit your crontab with `crontab -e`, and add the line `@reboot /root/ddns-updater.sh &>/dev/null`

#### Windows

3. Download and install [DDNS Updater](https://ddnsupdater.videocoding.org/download.html)

4. In the configuration window, enter your credentials, and set the IPv4 Update URL to "**https<nolink>://ydns<nolink>.io/api/v1/update/?host=example.ydns.eu**" (replacing "**example.ydns.eu**" with the name and domain you chose in step 2)

5. (Optional) Set the External check URL to "**https<nolink>://ydns<nolink>.io/api/v1/ip**"
