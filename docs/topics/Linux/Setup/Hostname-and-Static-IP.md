<!--
SPDX-FileCopyrightText: 2021 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Linux Setup: Hostname and Static IP

{{ j2_template_note }}

<!-- vim-markdown-toc GitLab -->

* [Set Hostname](#set-hostname)
  * [Systemd-based distros](#systemd-based-distros)
  * [Generic method](#generic-method)
* [Configure Static IP](#configure-static-ip)
  * [CentOS 7](#centos-7)
  * [Ubuntu Netplan](#ubuntu-netplan)
    * [FOR ETHERNET CONNECTIONS:](#for-ethernet-connections)
    * [FOR WIFI NETWORKS:](#for-wifi-networks)
  * [NetworkManager with nmcli](#networkmanager-with-nmcli)

<!-- vim-markdown-toc -->

## Set Hostname

### Systemd-based distros
{% raw %}
run `#!sh hostnamectl set-hostname {{ new_hostname }}`
{% endraw %}
Any currently logged-in users will need to log out and log back in to see the new hostname reflected.

### Generic method

replace the old hostname in `/etc/hosts` and `/etc/hostname`.

## Configure Static IP

> Used on Ubuntu Desktop and CentOS 8, among others

### CentOS 7

0. Run `ip link show`, and note the interface you are going to change. (In my case, it's **ens192**).

1. Edit the file `/etc/sysconfig/network-scripts/ifcfg-{{ IFNAME }}` to edit or add the following values:
{% raw %}
```
BOOTPROTO=none
ONBOOT=yes
IPADDR={{ IP_ADDR }}
PREFIX={{ PREFIX }}
GATEWAY={{ GATEWAY }}
DNS1={{ DNS_IP }}
```

* `{{ IFNAME }}`: name of the interface
* `{{ IP_ADDR }}`: the IP address
* `{{ PREFIX }}`: number of bits in the subnet mask
* `{{ GATEWAY }}`: the default gateway
* `{{ DNS_IP }}`: the IP address of the default gateway

Additionally, to specify a Search Domain, add the line `DOMAIN={{ SEARCH_DOMAIN_0 }}`
{% endraw %}
As a more concrete example, if you were setting up a static IP of **192.0.2.131** on the network **192.0.2.0/25**, with a default gateway of **192.0.2.129**, and a DNS server at **192.0.2.130**, and the search domain **for.example**, you would want to use the following values:
```
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.0.2.131
PREFIX=25
GATEWAY=192.0.2.129
DNS1=192.0.2.131
DOMAIN=for.example
```
2. Restart the network service with `systemctl restart network`.

### Ubuntu Netplan

> Starting with Ubuntu 18.04, networking can be configured with **netplan**, which will automatically configure networking based on the contents of a YAML file using either **NetworkManager** or **Systemd-networkd**.

0. escalate to root: `sudo -i`

1. get a list of network interfaces: `ip link show`

   * from this point on, I'll be using the interface **enp3s0** for ethernet and **wlp2s0** for wi-fi. If working with a different interface, replace any reference to either of them with the appropriate interface.

2. create a new YAML file in */etc/netplan/*.
   * For the sake of these notes, I'm naming mine **02-static-ensp3s0.yaml**, but any file ending in **.yaml** should work. It's best to start with a number, so that it's clear which are prioritised (**02-static-ensp3s0.yaml** would override any contradictory settings in an existing file called **01-old-configuration.yaml**)

3. using a text editor (e.g. `vim`, `nano`, or `gedit`), add the following to the YAML file you created, replacing `{{ jinja2_style_placeholders }}` with the appropriate values:

#### FOR ETHERNET CONNECTIONS:
{% raw %}
```yaml
network:
  version: 2
  renderer: "{{ RENDERER }}"
  ethernets:
    enp3s0:
      dhcp4: false
      dhcp6: false
      addresses:
        - "{{ IP_ADDRESS }}/{{ PREFIX }}"
      #gateway4: "{{ GATEWAY }}"
      routes:
        - to: default
        - via: "{{ GATEWAY }}"
      nameservers:
        search: ["{{ SEARCH }}"]
        addresses: ["{{ DNS_IP }}"]
```

#### FOR WIFI NETWORKS:

```yaml
network:
  version: 2
  renderer: "{{ RENDERER }}"
  wifis:
    wlp2s0:
      dhcp4: no
      dhcp6: no
      addresses:
        - "{{ IP_ADDRESS }}/{{ PREFIX }}"
      #gateway4: "{{ GATEWAY }}"
      routes:
        - to: default
        - via: "{{ GATEWAY }}"
      nameservers:
        search: ["{{ SEARCH }}"]
        addresses: ["{{ DNS_IP }}"]
      access-points:
        "{{ SSID }}":
          password: "{{ WPA2_PASS }}"
```

* `{{ RENDERER }}`: `networkd` or `NetworkManager`, depending on which you use.
  * **NetworkManager** is the default on Ubuntu Desktop, and **systemd-networkd** is the default for Ubuntu Server
* `{{ IP_ADDRESS }}`: the IP address of the server
* `{{ PREFIX }}`: the number of bits in the subnet mask
* `{{ GATEWAY }}`: the default gateway
* `{{ SEARCH }}`: search domains
* `{{ DNS_IP }}`: ip address/es of DNS Server/s
   * if you have multiple search domains or DNS servers,separate them with a comma followed by a space
* `{{ SSID }}`: the SSID (display name) of the network
* `{{ WPA2_PASS }}`: The WPA2 password for the network
{% endraw %}
**NOTE:** as of Ubuntu 22.04 Jammy Jellyfish, `gateway4` has been deprecated, and must be replaced with the `routes:` section. If working with an older version, either `gateway4` or a `routes:` section with a default route will work, but I'd recommend sticking with the `routes:` section, as it will make any future upgrades smoother.

4. test the configuration file with `netplan try`

* this will attempt to use your settings, and revert to a previous configuration if they don't work.

5. apply the netplan configuration: `netplan apply`

6. restart the network

   * if using **NetworkManager**: `systemctl restart NetworkManager`
   * if using **systemd-network**: `systemctl restart systemd-networkd`

### NetworkManager with nmcli

> If on Ubuntu, this assumes that netplan is set up to delegate control to **NetworkManager**, which is the default on Ubuntu Desktop, but not Ubuntu Server. If on Ubuntu Server, I'd recommend sticking with **netplan/systemd-networkd**

0. get a list of connections with `nmcli con show`
   * you can run `nmcli con show --active` to only display active connections

1. edit the active connection with the following command (replacing `{{ jinja2_style_placeholders }}` as needed)
{% raw %}
```sh
nmcli con mod "{{ CONNECTION }}" \
    ipv4.addresses "{{ IP_ADDRESS }}/{{ PREFIX }}" \
    ipv4.gateway "{{ DEFAULT_GATEWAY }}" \
    ipv4.dns "{{ DNS_IP_ADDRESS_0,DNS_IP_ADDRESS_1... }}" \
    ipv4.dns-search "{{ SEARCH_DOMAIN_0,SEARCH_DOMAIN_1... }}" \
    ipv4.method "manual"
```

2. restart the connection: `nmcli con down {{ CONNECTION }} && nmcli con up {{ CONNECTION }}`
{% endraw %}
