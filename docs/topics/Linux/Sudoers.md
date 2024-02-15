# Linux Sudoers

The rules for which users/groups can use sudo are typically defined in the file */etc/sudoers*. Typically, Linux distros will have a group which can use `sudo`, but the name of the group is not standardised across distros. Here are common default names for such groups:

## Default Sudoer Groups:

| Distro/s                                                                    | Group Name |
|-----------------------------------------------------------------------------|------------|
| **Red Hat Enterprise Linux**, **CentOS**, **Fedora**, and their derivatives | `wheel`  |
| **Debian**, **Ubuntu**, and their derivatives                               | `sudo`   |
| **Ubuntu** (11.10 and earlier)                                              | `admin`  |

**WARNING:** You should **never** edit **/etc/sudoers** directly - use the `visudo` command. (This will ensure that if your changes have any errors, they will be caught and reverted.)

## Specify Permissions:

### NOPASSWD

Allow the user **example** to use `sudo` without needing to enter a password by adding the following line to the end of the file

`example ALL=(ALL) NOPASSWD:ALL`

Instead of the user **example**, you can specify members of the group **example** by adding a percent sign to the beginning:

`%example ALL=(ALL) NOPASSWD:ALL`

### Allow a Specific Command:

Allow the user **example** to use **sudo** without a password, but only when running `reboot`, `halt`, or `poweroff`:

`example ALL=NOPASSWD: /sbin/reboot, /sbin/halt, /sbin/poweroff`

**WARNING:** Giving limited sudo access to a user can still be dangerous, depending on the commands you give access - this should be considered a convinience, not a security control, unless you ***really*** know what you're doing.
