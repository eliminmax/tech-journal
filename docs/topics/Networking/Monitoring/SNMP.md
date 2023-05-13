# SNMP on various systems

> <details>
> <summary>Meaning of different command prompts</summary>
> Unix/Linux: <code>$</code>: can be run as normal user<br>
> Unix/Linux: <code>#</code>: must be run as root (or with <code>sudo</code>)<br>
> Windows: <code>></code>: Command Prompt or PowerShell<br>
> Windows: <code>PS></code>: PowerShell only<br>
> Unix/Linux and Windows: <code>$/></code>,<code>#/></code>: Works in Windows and Unix/Linux.
> </details>

## CentOS Server

To enable SNMP on a CentOS server, install the net-snmp-utils and net-snmp packages with yum (`# yum install net-snmp-utils net-snmp -y`) and edit the configuration file at */etc/snmp/snmpd.conf* to suit your needs.

## Windows Server (Via AD DS)

To enable SNMP on a Windows server managed by AD DS, install the SNMP feature through **Server Manager**, and (if needed) install the associated Remote Administration Feature (i.e. **SNMP-Tools**) on the appropriate server. If needed, enable remote management.

* Note: To install the Remote Administration Feature locally as a Domain Admin, I had to run Server Manager as Administrator

 On the server you want to manage, allow Remote Management through the firewall: `PS> Set-NetFirewallRule -DisplayGroup "Remote Event Log Management" -Enabled True`

## pfSense Firewall/Router
To enable SNMP on a pfSense firewall/router, go to the web admin interface, and, from the Services drop-down, select ***SNMP***. Fill in the fields with the appropriate values, save, and click the restart service button at the top of the page
