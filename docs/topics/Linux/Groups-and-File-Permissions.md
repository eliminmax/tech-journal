<!--
SPDX-FileCopyrightText: 2020 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Linux Groups and File Permissions

<!-- vim-markdown-toc GitLab -->

* [Creating and modifying groups](#creating-and-modifying-groups)
  * [Creating groups](#creating-groups)
  * [Adding users to groups](#adding-users-to-groups)
  * [Removing users from groups](#removing-users-from-groups)
* [Switching primary group](#switching-primary-group)
* [File Permissions](#file-permissions)
  * [Changing owner and/or group](#changing-owner-andor-group)
  * [File mode](#file-mode)
    * [Examples](#examples)
    * [Changing file mode](#changing-file-mode)
    * [Special permission bits](#special-permission-bits)
      * [On Files](#on-files)
  * [Default permissions](#default-permissions)
    * [Querying and modifying the umask](#querying-and-modifying-the-umask)

<!-- vim-markdown-toc -->

These commands are important-to-know Linux commands for changing file permissions and managing groups.

## Creating and modifying groups

These commands must be run as the root user.

### Creating groups

`groupadd sample`: create a group called **sample**

### Adding users to groups

`usermod -a -G sample test`: add the user **test** to the group **sample**

### Removing users from groups

`gpasswd -d test sample`: remove the user **test** from the group **sample**

## Switching primary group

The `newgrp` tool can be used to launch a new shell with a different primary group. For example, `newgrp sample` will launch a new instance of the default login shell, but with the group set to `sample` - any new files created will have `sample` as the group. (I've mostly used this to switch to a group I was just added to without having to log out and log back in).

## File Permissions

### Changing owner and/or group

`chown test example.txt`: change the owner of **example.txt** to **test**. Must be run as root.

`chown test:sample example.txt`: change the owner of **example.txt** to **test**, and the group to **sample**.
* must be run as root, unless it's already owned by **test**, and **test** is the one running the command
* if run as root, **test** does not need to be a member of **sample**

`chgrp sample example.txt` or `chown :sample example.txt`: change the group of **example.txt** to **sample** 
* can be run by a normal user, as long as they are both the owner of **example.txt**, and a member of the group **sample**.
* if run as root, the owner does not need to be a member of **sample**.

### File mode

Every file has an owner and group, and three sets of permissions. One is for the owner, one is for members of the group, and one is for everyone else. These permissions are encoded into the "file mode".

Each set of permissions has three values: `r`, `w`, and `x`.

`r` is permission to **r**ead, `w` is permission to **w**rite, and `x` is permission to e**x**ecute. (NOTE: permission to "execute" a directory is needed to be able to `cd` into it.)

The overall permissions can be written with nine characters: `rwxrwxrwx` is full permissions, `---------` is no permissions, and they can be combined to create the appropriate permission scheme. The first three characters are the owning user's permissions, the next three are the group's permissions, and the last three are the permissions for anyone else.

It can also be shortened to a set of three octal numerals. **4** is read permissions, **2** is write permissions, and **1** is execute permissions, and add them to combine them (7 is all three, 5 is read+execute, 1 is execute, etc.)
 
#### Examples

| permissions | mode   | user r | user w | user x | group r | group w | group x | other r | other w | other x |
|-------------|--------|--------|--------|--------|---------|---------|---------|---------|---------|---------|
| `rwxr-x---` | `0750` | yes    | yes    | yes    | yes     | no      | yes     | no      | no      | no      |
| `rwxr-xr--` | `0754` | yes    | yes    | yes    | yes     | no      | yes     | yes     | no      | no      |
| `--x-w-rwx` | `0127` | yes    | yes    | no     | no      | yes     | no      | yes     | yes     | yes     |

#### Changing file mode

The standard tool to change file mode is `chmod`:

`chmod 754 example.sh`: changes the permissions for **example<nolink>.sh** to `rwxr-xr--` (anyone can read, user+group can execute, user can write)

`chmod` supports an alternative "symbolic mode" syntax:

`chmod u+x,g-w example.sh`: add execute permissions to owning **u**ser, revokes write permissions from **g**roup.

`chmod u=rwx,g=rx,o=r example.sh`: changes the permissions for **example<nolink>.sh** to `rwxr-xr--`

`chmod a+x example.sh`, `chmod +x example.sh`: allow **a**ll to execute

`chmod g=o` copies the permissions for **o**ther users into the **g**roup permissions.

#### Special permission bits

While often omitted and zeroed by default, the mode contains 3 extra bits before the user bits.
These are special permission bits: the `SETUID` (a.k.a. `SUID`) bit (octal 4000), `SETGID` (a.k.a. `SGID`) bit (octal 2000), and the sticky bit (octal 1000).

***WARNING:*** **These are very dangerous to mess around with - especially the `SETUID` bit. They must be used with caution, and are a common source of security issues - see [Penetration-Testing/SETUID-Binaries](../Penetration-Testing/SETUID-Binaries.md). They should be set sparingly, and only on executables explicitly designed to be safe to use with them, and those executables (and their dependencies) must be kept up-to-date to address any vulnerabilities they may have.**

They each have different behavior on normal files than they do on directories.

##### On Files

Normally, when running an executable, it runs with the permissions of the user that launched it, and the current primary group. The `SETUID` and `SETGID` bits change that - if an executable has the `SETUID` bit set, it instead runs as its owner. This is needed for a command like `passwd`, as it needs root-level permissions to be able to edit the `/etc/shadow` file. The `SETGID` bit does the same for the current group.

### Default permissions

The current user and primary group are the user and group of any new files or directories. 

When creating a new file, it defaults to taking the mode `0666` for files, or `0777` for directories, then removing any bits set in the current process's **umask** value - so with a **umask** of `0022`, any new files or directories will default to being world-readable but only writable by the current user, as in octal, `0666 &~ 0022` is `0644`, and `0777 &~ 0022` is `0755`. Different Linux systems will have different default **umask** values.

#### Querying and modifying the umask

In a shell enviromnment, the `umask` command with no arguments displays the current **umask** value in octal numeric form, and `umask -S` displays it in symbolic form.

If given an octal mask value as an argument, `umask` will change the current mask to that - so `umask 0077` will change the mask to block all users other than the owner from accessing any new files.
