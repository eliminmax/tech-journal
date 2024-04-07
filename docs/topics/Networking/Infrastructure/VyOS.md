<!--
SPDX-FileCopyrightText: 2022 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Linux: VyOS Router and Firewall

{{ j2_template_note }}

<!-- vim-markdown-toc GitLab -->

* [Configuration](#configuration)
  * [Entering Configuration Mode](#entering-configuration-mode)
  * [Save Configuration Changes](#save-configuration-changes)
  * [Basic system configuration](#basic-system-configuration)
    * [Set hostname](#set-hostname)
  * [Change password for user](#change-password-for-user)
  * [Configure SSH to use Key-based Authentication](#configure-ssh-to-use-key-based-authentication)
    * [The VyOS way](#the-vyos-way)
  * [Remote Syslog](#remote-syslog)
  * [Routing](#routing)
  * [Update VyOS (nightly)](#update-vyos-nightly)
  * [Firewall Management](#firewall-management)
    * [Define zones for interfaces](#define-zones-for-interfaces)
    * [Define Firewalls](#define-firewalls)
  * [Router Information Protocol (RIP)](#router-information-protocol-rip)
  * [Export/Import Configuration](#export-import-configuration)
  * [Port Forwarding](#port-forwarding)

<!-- vim-markdown-toc -->

## Configuration

*Much like [Cisco routers and switches](./Cisco-IOS.md), VyOS has a specific configuration mode, accessible with the `configure` command.*

### Entering Configuration Mode

Run the `configure` command

### Save Configuration Changes

In configuration mode, run `commit`, then run `save`.

To leave configuration mode, run `exit`.

### Basic system configuration

***Unless otherwise specified, run all of these in Configuration mode.***

#### Set hostname
{% raw %}
`set system host-name {{ hostname }}`

### Change password for user

`set system login user {{ username }} authentication plaintext-password {{ password }}`

* [the VyOS docs](https://docs.vyos.io/en/latest/configuration/system/login.html#local) explicitly state that passwords entered this way "will be automatically transferred into a secure hashed password and not saved anywhere in plaintext."
* if you'd rather not enter a visible plaintext password at all (and who could blame you), and have a hashed password available, you could replace the `plaintext-password {{ password }}` part of the command with `encrypted-password {{ hashed_password }}`

### Configure SSH to use Key-based Authentication

#### The VyOS way

(from the [VyOS Docs](https://docs.vyos.io/en/latest/configuration/system/login.html#ssh-key-based-authentication))

`set system login user {{ username }} authentication public-keys {{ key_name }} key {{ public_key }}`

### Remote Syslog

([from the VyOS Docs](https://docs.vyos.io/en/latest/configuration/system/syslog.html#cfgcmd-set-system-syslog-host-address-facility-keyword-level-keyword))

```
set system syslog host {{ address }} facility {{ keyword }} level {{ keyword }}
set system syslog host {{ address }} format {{ format }}
set system syslog host {{ address }} port {{ port }}
```

For a list of keywords to use, see [The VyOS Docs](https://docs.vyos.io/en/latest/configuration/system/syslog.html#facilities)

**For example:** send kernel debug logs to a [Gralyog](../Monitoring/Graylog.md) listener on 10.0.22.32 port 1514:

```
set system syslog host 10.0.222.32 facility kern level debug
set system syslog host 10.0.222.32 format octet-counted
set system syslog host 10.0.22.32 port 1514
```

### Routing

**For this example, I am using the interface eth0 connected to WAN, with the static IP address 203.0.113.5/24, and the default gateway 203.0.113.1, and Cloudflare's 1.1.1.1 DNS servers; and the interface eth1 connected to LAN, with the static IP address 10.10.40.1/24**

***YOU MUST `commit` AND `save` FOR CHANGES TO TAKE EFFECT***

0. Disable DHCP on interface (if necessary)
```
delete interfaces ethernet eth0 address dhcp
delete interfaces ethernet eth1 address dhcp
```

1. Set interface descriptions
```
set interfaces ethernet eth0 description WAN-IF
set interfaces ethernet eth1 description LAN-IF
```

2. Set up IP addresses
```
set interfaces ethernet eth0 address 203.0.113.5/24
set interfaces ethernet eth1 address 10.10.40.1/24
```

3. Set up default route and name server
```
set protocols static route 0.0.0.0/0 next-hop 203.0.113.1
set system name-server 1.1.1.1
```

4. NAT forwarding
```
set nat source rule 10 description "NAT from LAN to WAN"
set nat source rule 10 outbount-interface eth0
set nat source rule 10 source address 10.10.40.0/24
set nat source rule 10 translation address masquerade
```

5. DNS forwarding
```
set service dns forwarding listen-address 10.10.40.1
set service dns forwarding allow-from 10.10.40.0/24
set service dns forwarding system
```

### Update VyOS (nightly)

0. Copy the URL for a build at [vyos.net/get/nightly-builds/](https://vyos.net/get/nightly-builds/)

1. In VyOS, type the following, **without pressing enter**:

```
add system image
```

2. Paste in the URL, press enter, follow the defaults, and reboot

### Firewall Management

**For this section, I am building off of the setup I set up in the [Routing](#routing) section**

#### Define zones for interfaces

**DOING THIS WILL ENABLE A BLOCK-BY-DEFAULT POLICY. Without defining firewall rules, this will effectively shut down any network that relies on this system.**

```
set zone-policy zone WAN interface eth0
set zone-policy zone LAN interface eth1
```

#### Define Firewalls

0. Define two firewall rule sets - allow LAN to WAN connections by default, block and log WAN to LAN connections by default:

```
set firewall name LAN-to-WAN default-action allow
set firewall name WAN-to-LAN default-action drop
set firewall name WAN-to-LAN enable-default-log
```

1. Apply the new rules to the appropriate zones

```
set zone-policy zone WAN from LAN firewall name LAN-to-WAN
set zone-policy zone LAN from WAN firewall name WAN-to-LAN
```

2. Allow TCP and UDP connections to 10.10.40.21 port 25565 - we want our pals to be able to access our Minecraft server, after all.

*Disclaimer: This would require destination NAT, also known as port forwarding, in a real-world scenario, which I have added instructions for ([see below](#port-forwarding))*

```
set firewall name WAN-to-LAN rule 10 action accept
set firewall name WAN-to-LAN rule 10 destination address 10.0.40.21
set firewall name WAN-to-LAN rule 10 destination port 25565
set firewall name WAN-to-LAN rule 10 protocol tcp_udp
set firewall name WAN-to-LAN rule 10 description "Allow WAN access to Minecraft server on 10.10.40.21"
```

3. Allow WAN-to-LAN connections if initiated by LAN

Often data needs to be sent bidirectionally. Currently, the outbound data will be allowed, but the inboud replies won't be - this is little better than being completely disconnected from the internet. Let's fix it:
```
set firewall name WAN-to-LAN rule 1 action accept
set firewall name WAN-to-LAN rule 1 state established enable
```
***Don't forget to `commit` and `save`!***

### Router Information Protocol (RIP)

Say we wanted to connect a second private network (192.168.30.0/23, on eth0 on a new router) to the LAN used in previous examples, and configure routing between them. Here's how it would work:

To configure RIP, run the following on the original router:
```
set protocols rip interface eth1
# advertise the network 
set protocols rip network 10.10.40.0/24
```

Similarly, on the new router, run the following
```
set protocols rip interface eth0
set protocols rip network 192.168.30.0/23
```

### Export/Import Configuration

In configuration mode, you can run `save {{ path_to_file }}`, `save scp://{{ user }}:{{ pass }}@{{ host }}/{{ path_to_file }}`, `save sftp://{{ user }}:{{ pass }}@{{ host }}/{{ path_to_file }}`, or `save tftp://{{ host }}/file` to write the running config to that location instead of /config/config.boot. The remote host must have an SSH server or TFTP server running for those to work (see [The VyOS Docs](https://docs.vyos.io/en/latest/cli.html#cfgcmd-save)). You can also run `load {{ file }}` (in configuration mode) to load a saved configuration from a file, or any of the following:

* `load scp://{{ user }}:{{ password }}@{{ host }}/{{ path_to_file }}`
* `load sftp://{{ user }}:{{ passwd }}@{{ host }}{{ path_to_file }}`
* `load ftp://{{ host }}/{{ path_to_file }}`
* `load http://{{ host }}/{{ path_to_file }}`
* `load https://{{ host }}/{{ path_to_file }}`
* `load tftp://{{ host }}/{{ path_to_file }}`
{% endraw %}
Again, see [the VyOS Docs](https://docs.vyos.io/en/latest/cli.html?highlight=backup#cfgcmd-load-uri)

### Port Forwarding

Building on [the above scenario](#firewall-management), here's how to allow external access to a Minecraft server hosted on a private network:

Add the Destination NAT rules as follows

```
set nat destination rule 10 description "Port forward to Minecraft server"
set nat destination rule 10 inbound-interface eth1
set nat destination rule 10 destination port 25565
set nat destination rule 10 translation address 10.10.40.20
set nat destination rule 10 protocol tcp_udp
```

For more information, see [this VyOS Knowledgebase article on the topic](https://support.vyos.io/en/kb/articles/nat-principles)
