# CentOS: Installing Apache

<details>
> <summary>Meaning of different command prompts</summary>
> Unix/Linux: <code>$</code>: can be run as normal user<br>
> Unix/Linux: <code>#</code>: must be run as root (or with <code>sudo</code>)<br>
> Windows: <code>></code>: Command Prompt or PowerShell<br>
> Windows: <code>PS></code>: PowerShell only<br>
> Unix/Linux and Windows: <code>$/></code>,<code>#/></code>: Works in Windows and Unix/Linux.
> </details>

To set up Apache, follow these simple steps:

0. Install the Apache software by running the following command, and confirm installation: `# yum install httpd`

1. Start the httpd service, and set it to run at boot: `# systemctl start httpd && systemctl enable httpd`

2. Allow the http and https services through the firewall: `# firewall-cmd --add-service=http --add-service=https --permanent && firewall-cmd --reload`
