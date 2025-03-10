<!--
SPDX-FileCopyrightText: 2021 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

This page will introduce the most basic uses of the `git` version control system

## Installing Git

### Debian, Ubuntu, and derivatives:


* On modern systems that support the `apt` command, run `apt install -y git` as root.

* On older systems that don't support the `apt` command, run `apt-get install -y git` as root.

### CentOS, RHEL, and derivatives:

If you don't have **git** installed already, run `yum -y install git` as root.

---

### Windows:

0. Go to https://git-scm.com/download, and click the Windows icon.

1. Run the installer

## Basic Git commands

### Clone a repository:

#### Clone via ssh:

`git clone user@host-or-ip:path/to/repository.git`

#### Clone via https:

`git clone https://host-or-ip/path/to/repository.git`

### Pull changes from origin:

`git pull`

### Download file from origin:

`git checkout .`

> Useful if a file is deleted or overwritten by mistake

### Add local file to repository

> Even if a file is added to your local copy of a repository, it won't be added to the git repo automatically. To add it, you need to run this command.

`git add filename.ext`

### Commit changes

#### Commit all changes

`git commit`

#### Commit changes to specific files:

`git commit file0.ext file1.ext file2.ext`

#### Specify commit message in command:

`git commit -m "Commit message"`

You can combine this with the previous one:

`git commit -m "Commit message" file0.ext file1.ext file2.ext`

### Push local changes to origin:

`git push`

> Changes must be committed for this to go through.
