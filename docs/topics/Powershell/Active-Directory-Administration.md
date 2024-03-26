<!--
SPDX-FileCopyrightText: 2021 - 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Active Directory Administration via Powershell

<!-- vim-markdown-toc GitLab -->

* [Install and Configure Domain Controller and DNS Server](#install-and-configure-domain-controller-and-dns-server)
  * [Set up AD DS Forest](#set-up-ad-ds-forest)
  * [Add DNS Records](#add-dns-records)
  * [Print DNS Records](#print-dns-records)
* [DHCP](#dhcp)
* [Organizational Units](#organizational-units)
  * [Create an Organizational Unit](#create-an-organizational-unit)
  * [Delete an Organizational Unit](#delete-an-organizational-unit)
  * [Move an Object into an Organizational Unit](#move-an-object-into-an-organizational-unit)
* [Miscellaneous](#miscellaneous)
  * [Enable Remote Management](#enable-remote-management)
  * [Parsing Event Logs](#parsing-event-logs)

<!-- vim-markdown-toc -->

## Install and Configure Domain Controller and DNS Server

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

### Set up AD DS Forest

```powershell
Install-ADDSForest -DomainName 'for.example'
```

### Add DNS Records

```powershell

# Create lookup zone
Add-DnsServerPrimaryZone -NetworkID '192.0.2.0/24' -ReplicationScope 'Domain'

# Add A record and associated PTR
Add-DnsServerResourceRecordA -Name 'host' -ZoneName 'for.example' -IPv4Address '192.0.2.201' -CreatePtr

# Add PTR Record
Add-DnsServerResourceRecordPTR -Name '12' -ZoneName '1.0.192.in-addr.arpa.' -PtrDomainName 'gateway'
```

### Print DNS Records

To print all DNS type A records stored on the AD DNS Server, run `Get-DnsServerResourceRecord -ZoneName <AD DS DNS Zone> -ComputerName <AD DS DNS Server> -RRType A`

(e.g. for a domain in the network for.example, with a DNS server at dns01.for.example, it would be `Get-DnsServerResourceRecord -ZoneName for.example -ComputerName dns01 -RRType A`)

For reverse lookup zones, you can run the same command, but instead of `-RRType A`, you'd need to specify `-RRType Ptr`, and `-ZoneName` would need to be the reverse lookup zone.

(e.g. for the same domain as before, if the network was at 192.0.2.0/24, the command would be `Get-DnsServerResourceRecord -ZoneName 2.0.192.in-addr.arpa -ComputerName dns01 -RRType PTR`)

## DHCP

To install the needed component and create a DHCP scope, run the following

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DHCPServerv4Scope -Name 'for.example DHCP scope' -StartRange 192.0.2.50 -EndRange 192.0.2.75 -SubnetMask 255.255.255.0
```

## Organizational Units

### Create an Organizational Unit

Use the `New-ADOrganizationalUnit` cmdlet: `New-ADOrganizationalUnit -Name 'Example OU' -Path 'DC=for,DC=example'`

   * Read more at the [PowerShell Docs](https://docs.microsoft.com/en-us/powershell/module/activedirectory/new-adorganizationalunit?view=windowsserver2019-ps)

### Delete an Organizational Unit

Use the `Remove-ADOrganizationalUnit` cmdlet: `Remove-ADOrganizationalUnit -Identity 'OU=Example OU,DC=for,DC=example'`

   * Read more at the [PowerShell Docs](https://docs.microsoft.com/en-us/powershell/module/activedirectory/remove-adorganizationalunit?view=windowsserver2019-ps)

### Move an Object into an Organizational Unit

Use the `Move-ADObject` cmdlet: `Move-ADObject -Identity 'CN=Example User,CN=Users,DC=for,DC=example'`

   * Read more at the [PowerShell Docs](https://docs.microsoft.com/en-us/powershell/module/activedirectory/move-adobject?view=windowsserver2019-ps)

## Miscellaneous

### Enable Remote Management

```powershell
Enable-NetFirewallRule -DisplayGroup 'Windows Remote Management'
```

### Parsing Event Logs

Use the `Get-WinEvent` cmdlet

  * Read more at the [PowerShell Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1)
