<!--
SPDX-FileCopyrightText: 2023 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Ansible: Installation Methods

## Install Ansible with System Package Manager

A list of distro repos and the versions they ship is available on [Repology](https://repology.org/project/ansible/versions).

### Red Hat Family

If on a supported version on Red Hat Enterprise Linux or a "bug-for-bug compatible" distro like CentOS, Rocky Linux, or AlmaLinux, you should first enable EPEL. On Fedora systems, skip this step.

```sh
# For CentOS/RHEL 7, etc.
sudo yum install epel-release
# For Rocky Linux/AlmaLinux/RHEL 8 and newer, etc.
sudo dnf install epel-release
```

After doing that, you can install the `ansible` package.

```sh
# For CentOS/RHEL 7, etc.
sudo yum install ansible
# For Fedora, CentOS Stream, Rocky Linux/AlmaLinux/RHEL 8 and newer, etc.
sudo dnf install ansible
```

### Debian Family

The simplest way to do this would be the following:

```sh
sudo apt install ansible
```

In my personal experience, that should be fine in most cases, but it does install an older version of ansible. There are a few options if you need the latest version - you can install from a PPA (recommended for Ubuntu-based distros), install with `pip`, install with `pipx`, or install with `spipx`.

#### Install from PPA

The instructions from [the Ansible docs](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu) say to install from the PPA on Ubuntu-based systems.

```sh
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

[The Ansible docs](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian) include instructions for using the PPA on Debian as well, but I'd highly recommend against that approach, because it uses the deprecated `apt-key` command, and it is considered bad practice to use PPAs on Debian - see [Advice For New Users On Not Breaking Their Debian System § Don't make a FrankenDebian](https://wiki.debian.org/DontBreakDebian#Don.27t_make_a_FrankenDebian) on the Debian wiki for an explanation.

## Installation with pip and Related Tools

### pip

Assuming you have Python (version 3.9-3.11) with pip installed, the simplest way to install is as follows:
```sh
python3 -m pip install --user ansible
```

### pipx

`pipx` is a tool that, much like `pip`, can install programs from the Python Package Index, but unlike `pip`, it installs them into their own isolated venv, to avoid polluting the system Python prefix and avoid dependency hell.

```sh
pipx install ansible --include-deps
```

*Note: without the --include-deps flag, it would still install ansible and its dependencies to the venv, but would only add `ansible-core` to your PATH, not the full suite of commands.*

Installing pipx can be done a few different ways:

#### Install pipx with System Package Manager

pipx is available for most modern distros, but might need some extra work to get it up and running.

On RHEL 9 and its bug-for-bug compatible derivatives, it's available through EPEL. On versions 8 and oolder, it is not available.

On Debian Bullsey (stable at time of writing), it is available through the backports repository. See [Debian Backports » Instructions](https://backports.debian.org/Instructions/) if you need instructions to enable it.

#### Install pipx with pip

```sh
python3 -m pip install --user pipx
```

I'd recommend against this method, as installing as user is a recipe for dependency hell, while installing as root is a recipe for dependency hell that can break your system. See the [pipx-in-pipx](#install-pipx-with-pipx-in-pipx) instructions below for an alternative.

#### Install pipx with pipx-in-pipx

`pipx-in-pipx` is a tool that bootstraps `pipx`, by installing it into an isolated venv that `pipx` will then manage - so you'll have a self-managing `pipx` installation. When installing a package, `pip` runs its `setup.py` script. The `pipx-in-pipx` `setup.py` script simply runs through the bootstrapping process I described, then uninstalls itself, leaving no other traces on the system. It is hacky, but results in a cleaner python setup, and helps prevent your laptop from becoming a superfund site - see [xkcd #1987](https://xkcd.com/1987/).

```sh
python3 -m pip install --user pipx-in-pipx
```

### spipx

This is a script of my own creation, which builds on `pipx`. Normally, pipx will install things to ~/.local/pipx, and add symlinks to the executables it manages in ~/.local/bin. Those paths can be overridden with the `PIPX_HOME` and `PIPX_BIN_DIR` environment variables, respectively.

I figured that if multiple users would be using `pipx`, it might be better to set `pipx` to install things to a directory shared across users. The way I do that is with the following:

```sh
sudo dd of=/usr/local/sbin/spipx <<EOF
#!/bin/sh

if [ "\$(id -u)" -ne 0 ]; then
    printf 'Error: spipx must be run as root!\n'
fi

PIPX_HOME=/opt/pipx PIPX_BIN=/opt/pipx/bin pipx "\$@"
EOF
sudo chmod +x /usr/local/sbin/spipx
```

Additionally, the PATH variable must include the /opt/pipx/bin directory. One way to make sure that that is the case is to run the following:

```sh
sudo dd of=/etc/profile.d/spipx_path.local.sh <<EOF
PATH="\$PATH\${PATH+:}/opt/pipx/bin"
EOF
. /etc/profile.d/spipx_path.local.sh
```

Finally, run the command

```sh
sudo spipx install ansible --include-deps
```
