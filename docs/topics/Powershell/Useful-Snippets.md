# Useful PowerShell Snippets

<!-- vim-markdown-toc GitLab -->

* [Windows Exclusive](#windows-exclusive)
  * [Check IP Addresses](#check-ip-addresses)
  * [Enable RDP](#enable-rdp)
* [Cross-platform](#cross-platform)
  * [Filter objects](#filter-objects)
  * [Install Module](#install-module)

<!-- vim-markdown-toc -->

# Windows Exclusive

## Check IP Addresses

```powershell
Get-NetIPAddress
```

## Enable RDP

```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

# Cross-platform

*Note: not all examples are cross-platform, but the concepts they are being used to illustrate are.*

## Filter objects

example: get the ID and Name of all processes that have been running since before midnight on 19 Jan. 2023

```powershell
Get-Process | Where-Object {$_.StartTime -lt (Get-Date '2023-01-29 00:00')} | Select-Object Id,Name
```

## Install Module

For the sake of example, I'll demonstrate the installation of VMware's PowerCLI toolset

```powershell
Install-Module -Name VMware.PowerCLI
```
