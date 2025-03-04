<!--
SPDX-FileCopyrightText: 2023 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Useful PowerShell Snippets

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
