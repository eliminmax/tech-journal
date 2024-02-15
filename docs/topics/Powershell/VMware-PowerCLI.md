# VMware PowerCLI

<!-- vim-markdown-toc GitLab -->

* [Overview](#overview)
  * [Connect to Server](#connect-to-server)
  * [List VMs](#list-vms)
  * [Start/Stop VMs](#start-stop-vms)
  * [Create Snapshot](#create-snapshot)
  * [Create Linked Clone From Snapshot](#create-linked-clone-from-snapshot)
  * [Create a Virtual Network](#create-a-virtual-network)
  * [Create a Fresh VM with an OS Installer ISO](#create-a-fresh-vm-with-an-os-installer-iso)

<!-- vim-markdown-toc -->

## Overview

PowerCLI is a Powershell module used to manage VMware ESXi and VCenter servers. To start, [install it](./Useful-Snippets.md#install-module), then run the following to configure it:

```powershell
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -ParticipateInCeip $False
```

This will ignore invalid certificates, which VMware servers have out-of-the-box, and disable the "Customer Experience Improvement Program" telemetry.

Note that there's a variety of ways to reference VMs, and I switch between them throughout this document, to help illustrate them. In general, it's a good idea to stick to a specific style.

### Connect to Server

```powershell
$powercli_creds = Get-Credentials
Connect-VIServer -Server 'esxi.for.example' -Protocol 'https' -Credential $powercli_creds
```

### List VMs

Once connected to a server, you can run `Get-VM` to get a list of VMs, optionally restricting them to specific criteria, like location (`-Location`), or tags (`-Tag`), etc.

### Start/Stop VMs

The `Start-VM` and `Stop-VM` cmdlet are used for this purpose:

```powershell
$foo = Get-VM -Name foo
Start-VM -VM $foo
Stop-VM -VM $foo
```

### Create Snapshot

```powershell
Get-VM -Name foo | New-Snapshot -Name "Foo-Snapshot-$(Get-Date -Format FileDateTimeUniversal)"
```

### Create Linked Clone From Snapshot

```powershell
New-VM -Name LinkedToBaz -LinkedClone -ReferenceSnapshot (Get-Snapshot -Name 'To-Clone' -VM 'Baz') -VM 'Baz'
```

### Create a Virtual Network

This one has 2 steps - Create the virtual switch, and create the port group associated with it.
```powershell
$NetworkName = 'Foonet'
$vswitch = New-VirtualSwitch -Name $NetworkName -VMHost '192.0.2.127'
$vportgrp = New-VirtualPortGroup -Name $NetworkName -VirtualSwitch $vswitch
```

### Create a Fresh VM with an OS Installer ISO

To start with, you'll want to know a few things about the VM, including the guest OS identifier PowerCLI uses.

Per [this answer on VMware's VMTN Comminities](https://communities.vmware.com/t5/VMware-PowerCLI-Discussions/How-to-get-all-VirtualMachineGuestOsIdentifier-over-PowerCLI/m-p/1397692/highlight/true#M45696), you can get a list of all valid values for this with the following:

```powershell
[VMware.Vim.VirtualMachineGuestOsIdentifier].GetEnumValues()
```

Note that this list is *long* - on my setup, it has 195 entries, though I suspect that varies depending on the exact version of PowerCLI. I'd recommend piping the above command into `Select-String` to filter down to what you actually want - e. g. for an Ubuntu guest, I might run the following:

```powershell
[VMware.Vim.VirtualMachineGuestOsIdentifier].GetEnumValues() | Select-String 'ubuntu'
```

If you are installing from an ISO file on the datastore, you will also want the path to the ISO file in question.

The way I found that was a bit convoluted, and as someone used to the Unix-like approach to file-systems, I found it surprising. For context, I knew that within my environment, I stored ISO files in a directory called `isos` on `datastore2`, and that the ISO file I wanted was the only one in the directory with `ubuntu` at the start of its name. With that knowledge, I got the path to an ISO file with the following:

```powershell
$ds2 = Get-Datastore -Name 'datastore2'
$ISO = (Get-ChildItem ($ds2.DatastoreBrowserPath) | Where-Object {$_.Name -eq 'isos'} | Get-ChildItem | Where-Object {$_.Name.StartsWith('ubuntu')})
```

With that knowledge, and those variables defined, you can then use the following, changing the values as appropriate:

```powershell
$NewVMParams = @{
  'Name' = 'ubuntu.22.04.server.base'
  'VMHost' = '192.168.7.13'
  'Datastore' = 'datastore1'
  'DiskGB' = 24
  'DiskStorageFormat' = 'Thin'
  'MemoryGB' = 3
  'GuestId' = 'ubuntu64Guest'
  'HardwareVersion' = 'vmx-17'
  'NumCpu' = 2
}
$VM = New-VM @NewVMParams

$NewCDDriveParams = @{
  'VM' = $VM
  'IsoPath' = $ISO.DatastoreFullPath
  'StartConnected' = $true
}
New-CDDrive @NewCDDriveParams
New-NetworkAdapter -NetworkName '480-WAN' -VM $VM
Start-Sleep 10s
Start-VM -VM $VM
```

Note that the above snippet was based on [this serverfault question](https://serverfault.com/q/891430), replacing the deprecated `Version` field with `HardwareVersion`, which is its more modern counterpart, and changing the field values to those I actually used for my class lab. I also replaced `Sleep` with `Start-Sleep`, as the former is an alias and I'd rather use the full form in a script or in documentation and notes. I also omitted the `Notes` parameter for the `New-VM` cmdlet.

Once the OS was installed, I shut off the VM and removed the ISO using the following 2 cmdlets:

```powershell
Stop-VM -VM $VM -Confirm:$false
Get-CDDrive -VM $VM | Set-CDDrive -NoMedia -Confirm:$false
```
