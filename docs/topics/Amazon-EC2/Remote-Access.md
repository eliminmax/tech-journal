<!--
SPDX-FileCopyrightText: 2021 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Amazon EC2: Access Machines Remotely

{{ j2_template_note }}

## Accessing a Linux Instance via SSH
{% raw %}
This one is simple, assuming that you have the identity file saved locally. Simply run the command `ssh {{ username }}@{{ ec2host }} -i {{ path_to_identity_file }}`
{% endraw %}
### Adding EC2 instances to SSH config

#### Method 1 (not recommended):

 To avoid needing to specify the identity file every time, assuming that you only use one identity file for all EC2 insrances, you can use my hacked-together script found [here](https://github.com/eliminmax/mini-utils/blob/main/ec2-ssh-config.sh).

<details>
<summary><em>Why use this script?</em></summary>
There are a few intersecting problems at play that prevent this from being a straightforward process: Amazon EC2 instances do not have static IP addresses, and every time they stop and start again, they have a new IP address any of several hundred different IP ranges, so adding a Host to your SSH config doesn't work. OpenSSH does not play nice with CIDR notation in its config files, so I adapted [this technique for using Match in SSH config files](https://serverfault.com/a/1043429) to use a list of addresses. Getting the list requires parsing a JSON file provided by Amazon, filtering out the non-EC2 IP ranges, and removing any excess data, and saving the result to a file. The script just does that automatically, rather than making you deal with it yourself.
</details>
<details>
<summary><em>Why not use this script?</em></summary>
It is a fragile mess that I hacked together without a clear understanding of what I was doing, and there are much, <em>much</em> better solutions, like the next one listed.
</details>

1. Install `jq` and `grepcidr` (on Debian/Ubuntu systems, run `apt install jq grepcidr -y` as `root`/with `sudo`)

2. Download the script: `wget https://raw.githubusercontent.com/eliminmax/mini-utils/main/ec2-ssh-config.sh`

3. Read over the script, and ensure that you can trust it.

4. Mark it as executable with `chmod u+x ec2-ssh-config.ssh`

5. Run the script with `./ec2-ssh-config.ssh`, and enter the path to the identity file.

#### Method 2 (recommended):

1. Set up Dynamic DNS with DuckDNS (see [Networking: Infrastructure: Dynamic DNS](../Networking/Infrastructure/Dynamic-DNS.md))

2. Edit or create user-specific ssh config (located at `~/.ssh/config` on Unix-like systems (including macOS and Linux), and `%userprofile%\.ssh.\config` on Windows), adding the following:
{% raw %}
```sshconfig
Host {{ shortname }}
	Hostname {{ subdomain }}.duckdns.org
	IdentityFile {{ path_to_identity_file }}
```
{% endraw %}
## Accessing a Windows Server instance with RDP

Ensure that the EC2 firewall is configured to allow Inbound RDP traffic.

see [Amazon EC2: Firewall Management](./Firewall-Management.md) and [Amazon EC2: Get Default Windows Password](./Get-Default-Windows-Password.md)

### On Windows

Use Remote Desktop Connection. Enter the public IP address of the instance, and when it asks for your credentials, enter them.

### On Linux

I'd recommend [Remmina](https://remmina.org/), with the Remmina RDP plugin. From the **Remmina Remote Desktop Client Window**, choose **RDP** in the drop-down, and enter the public IP address of the instance. Enter the credentials when prompted.
