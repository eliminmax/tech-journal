<!--
SPDX-FileCopyrightText: 2020 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Linux Groups and File Permissions

<!-- vim-markdown-toc GitLab -->

* [Creating and modifying groups](#creating-and-modifying-groups)
  * [groupadd](#groupadd)
  * [usermod -a -G](#usermod-a-g)
  * [gpasswd -d](#gpasswd-d)
* [Changing file ownership](#changing-file-ownership)
  * [chown](#chown)
* [File permissions](#file-permissions)
  * [Examples](#examples)
  * [chmod](#chmod)

<!-- vim-markdown-toc -->

These commands are important-to-know Linux commands for changing file permissions and managing groups.
Except where otherwie noted, they must be run as root.

## Creating and modifying groups

### groupadd

`groupadd sample`: create a group called **sample**

### usermod -a -G

`usermod -a -G sample test`: add the user **test** to the group **sample**

### gpasswd -d

`gpasswd -d test sample`: remove the user **test** from the group **sample**

## Changing file ownership

### chown

`chown test example.txt`: change the owner of **example.txt** to **test**

`chown test:sample example.txt`: change the owner of **example.txt** to **test**, and the group to **sample**

* **test** does not need to be a member of **sample**

`chown :sample example.txt`: change the group of **example.txt** to **sample** 

* can be run by a normal user, as long as they are both the owner of **example.txt**, and a member of **sample**.

* root or the file's owner can also run `chgrp sample example.txt` for the same effect, with the same restrictions as `chown :sample example.txt` for the owner

* if run as root, the owner does not need to be a member of **sample**.

## File permissions

Every file has an owner and group, and three sets of permissions. One is for the owner, one is for members of the group, and one is for everyone else.

Each set of permissions has three values: `r`, `w`, and `x`.

`r` is permission to **r**ead, `w` is permission to **w**rite, and `x` is permission to e**x**ecute.

The overall permissions can be written with nine characters: `rwxrwxrwx` is full permissions, `---------` is no permissions, and they can be combined to create the appropriate permission scheme. The first three characters are the owning user's permissions, the next three are the group's permissions, and the last three are the permissions for anyone else.

It can also be shortened to a set of three octal numerals. **4** is read permissions, **2** is write permissions, and **1** is execute permissions, and add them to combine them (7 is all three, 5 is read+execute, 1 is execute, etc.)
 
### Examples

| permissions | numeric | user r | user w | user x | group r | group w | group x | other r | other w | other x |
|-------------|---------|--------|--------|--------|---------|---------|---------|---------|---------|---------|
| `rwxr-x---` | 750     | yes    | yes    | yes    | yes     | no      | yes     | no      | no      | no      |
| `rwxr-xr--` | 754     | yes    | yes    | yes    | yes     | no      | yes     | yes     | no      | no      |
| `--x-w-rwx` | 127     | yes    | yes    | no     | no      | yes     | no      | yes     | yes     | yes     |

### chmod

These commands do not require root privileges

`chmod 754 example.sh`: changes the permissions for **example<nolink>.sh** to `rwxr-xr--` (anyone can read, user+group can execute, user can write)

`chmod u+x,g-w example.sh`: grants execute permissions to the **u**ser, revokes write permissions from **g**roup.

`chmod u=rwx,g=rx,o=r example.sh`: changes the permissions for **example<nolink>.sh** to `rwxr-xr--`

`chmod a+x example.sh`, `chmod +x example.sh`: allow **a**ll to execute
