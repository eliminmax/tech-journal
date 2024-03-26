<!--
SPDX-FileCopyrightText: 2020 - 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Active Directory: Windows DHCP Server

0. In Server Manager, add the **DHCP** role to the server you intend to use for DHCP, and add **DHCP Server Tools** feature

1. A DHCP tab will appear on the sidebar of Server Manager. Switch over to it.

2. There will be a notice that reads "⚠️ Configuration required for DHCP Server at <hostname>". Click "More..."

3. Click "Complete DHCP configuration" in All Servers Task Details and Notifications, and authorize a user to manage it.

4. In Server Manager, Go to the tools Menu, and open the DHCP configuration window

5. In the Action menu, go to `Manage Authorized Servers` and add the DHCP Server to the console.

6. Right Click `<DHCP server> > IPv4`and add a scope, and configure the range of IP addresses it's able to offer, and the network settings (Default Gateway, Domain, and DNS). Save the configuration and deploy your DHCP.

> Source: [Install and Configure DHCP Server on Windows Server 2019 | ComputingForGeeks](https://computingforgeeks.com/how-to-install-and-configure-dhcp-server-on-windows-server/)
