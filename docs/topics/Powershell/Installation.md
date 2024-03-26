<!--
SPDX-FileCopyrightText: 2023 - 2024 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Powershell Installation

<!-- vim-markdown-toc GitLab -->

* [Windows](#windows)
* [Linux](#linux)
  * [Debian 11 (Bullseye)](#debian-11-bullseye)

<!-- vim-markdown-toc -->

## Windows

Windows comes bundled with Windows Powershell.

## Linux

Microsoft provides [instructions for several distros](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux?view=powershell-7.3), which should work in a pinch. I don't follow them directly, but it's still a good guide.

### Debian 11 (Bullseye)

The official instructions say to run the following commands:

```sh
# Install system components
sudo apt update  && sudo apt install -y curl gnupg apt-transport-https

# Import the public repository GPG keys
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -

# Register the Microsoft Product feed
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-bullseye-prod bullseye main" > /etc/apt/sources.list.d/microsoft.list'

# Install PowerShell
sudo apt update && sudo apt install -y powershell

# Start PowerShell
pwsh
```

I have a few issues with this: the `apt-key` command is deprecated, and the `apt-transport-https` package is now an empty dummy package, and according to the
output of `apt show apt-transport-https`, its functionality has been merged into `ap`t itself. The reason `apt-key` is deprecated is that it trusts the gpg key to sign all packages, not just those in a particular repo. Because of that, I would do the following:

```sh
# Install system components
sudo apt update && sudo apt install -y curl gnupg

# Import the public repository GPG keys
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo dd of=/etc/apt/keyrings/microsoft.gpg

# Register the Microsoft Product feed
sudo dd of=/etc/apt/sources.list.d/microsoft.list <<EOF
https://packages.microsoft.com/repos/microsoft-debian-bullseye-prod bullseye main
EOF

# Install PowerShell
sudo apt update && sudo apt install -y powershell

# Start PowerShell
pwsh
```

This does not rely on deprecated functionality or dummy packages, and has the advantage of only trusting the signing key for packages in the Microsoft apt repository.
