<!--
SPDX-FileCopyrightText: 2021 - 2024 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# SNMP on various systems

<!-- vim-markdown-toc GitLab -->

* [CentOS Server](#centos-server)
* [Windows Server (Via AD DS)](#windows-server-via-ad-ds)
* [pfSense Firewall/Router](#pfsense-firewall-router)

<!-- vim-markdown-toc -->

## CentOS Server

To enable SNMP on a CentOS server, install the net-snmp-utils and net-snmp packages with yum (`yum install net-snmp-utils net-snmp -y` as root or with sudo) and edit the configuration file at */etc/snmp/snmpd.conf* to suit your needs.

## Windows Server (Via AD DS)

To enable SNMP on a Windows server managed by AD DS, install the SNMP feature through **Server Manager**, and (if needed) install the associated Remote Administration Feature (i.e. **SNMP-Tools**) on the appropriate server. If needed, enable remote management.

* Note: To install the Remote Administration Feature locally as a Domain Admin, I had to run Server Manager as Administrator

On the server you want to manage, allow Remote Management through the firewall with PowerShell: `Set-NetFirewallRule -DisplayGroup "Remote Event Log Management" -Enabled True`

## pfSense Firewall/Router

To enable SNMP on a pfSense firewall/router, go to the web admin interface, and, from the Services drop-down, select ***SNMP***. Fill in the fields with the appropriate values, save, and click the restart service button at the top of the page
