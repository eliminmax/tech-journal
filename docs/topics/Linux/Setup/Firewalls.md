<!--
SPDX-FileCopyrightText: 2022 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Linux: Setup: Firewalls

<!-- vim-markdown-toc GitLab -->

* [Firewalld](#firewalld)
  * [Show active firewall config](#show-active-firewall-config)
  * [Add a port](#add-a-port)
  * [Add a service](#add-a-service)
  * [Add a rich rule](#add-a-rich-rule)

<!-- vim-markdown-toc -->

## Firewalld

**Note:**

By default, `firewall-cmd` makes changes to the running config, rather than the saved config.
If it is called with the `--permanent` flag, it does the opposite - it changes the saved
config, but not the running config. To update both, call it with the `--permanent` flag, then
call `# firewall-cmd --reload` to reload from the saved config, or call the command twice,
once with the `--permanent` flag, and once without. The latter method is usually better, as
the former will cause you to lose any deliberately-introduced differences between the running
and saved configs.
Additionally, `firewall-cmd` requires root privileges.

### Show active firewall config

`firewall-cmd --list-all`

### Add a port

Allow inbound tcp connections to port 22:

`firewall-cmd --add-port=22/tcp`

### Add a service

`firewall-cmd --add-service=ssh`

### Add a rich rule

A few examples modified from [ComputerNetworkingNotes](https://www.computernetworkingnotes.com/linux-tutorials/firewalld-rich-rules-explained-with-examples.html)

`firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.0.2.0/24 service name=ssh log prefix="SSH Access" level="notice" accept'`

`firewall-cmd --add-rich-rule='rule protocol value=icmp reject'
