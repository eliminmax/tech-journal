<!--
SPDX-FileCopyrightText: 2020 - 2022 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# CentOS: Set Up DHCP Service

To set up the DHCP service on CentOS, do the following:

0. Install dhcp client (as root / with `sudo`): `yum install dhcp`.

1. Edit the dhcp config file (**/etc/dhcp/dhcpd.conf**). Add the following, filling in the blanks. Anything in [square brackets] is optional:

```
subnet ___.___.___.___ netmask ___.___.___.___ {
    option routers ___.___.___.___;
    option subnet-mask ___.___.___.___;
    [option domain-name "________.___";]
    option domain-name-servers ___.___.___.___[, ___.___.___.___];
    range ___.___.___.___ ___.___.___.___;
}
```

**Example**: if you had a network **192.0.2.0/24**, with a router at **192.0.2.1**, and the domain **for<nolink>.example**, and you wanted to reserve addresses **192.0.2.1** through 192.0.2.127 for static IP addresses, and you had a DNS server at **192.0.2.2**, you'd want the following configuration:

```
subnet 192.0.2.0 netmask 255.255.255.0 {
    option routers 192.0.2.1;
    option subnet-mask 255.255.255.0;
    option domain-name "for.example";
    option domain-name-servers 192.0.2.0/24;
    range 192.0.2.128 192.0.2.254;
}
```
2. Start the service by running the command `systemctl start dhcpd`, and set it to start at boot with `systemctl enable dhcpd`.

3. Allow the DHCP service through the firewall with the command `firewall-cmd --add-service=dhcp --permanent`, then reload the firewall with `firewall-cmd --reload`.
