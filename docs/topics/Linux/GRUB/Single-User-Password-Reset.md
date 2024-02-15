# Single-user mode password reset 'hack' with GRUB

## Brief Technical Explanation

Operating systems need to be loaded by bootloaders at, well, boot time, and a popular bootloader on Linux is GRUB - the GRand Unified Bootloader.

GRUB and Linux, working in tandem, have a feature that allows one to boot into a root shell without a password - and I do mean a feature. Note that it does not work on encrypted installations, so encrypt any system you don't want this to work on.

## Executing the 'hack'

While this feature has several valid uses, here, we're going to use it to reset the root password.

0. If the system is running, shut it down.
1. When the system is booting up, wait for the GRUB menu to appear, then immediately type the letter e to edit the menu entry.
2. Using the arrow keys, navigate to the end of the line that starts with `linux` (this line might be indented) and append the word `single` to it to enable single-user mode.
3. The default init program (i.e. the first process that runs at startup) is probably not what we'll want - we want a shell. If we know that the GNU Bourne-Again SHell is installed to `/bin/bash`, we can pass the argument `init=/bin/bash` to use it as the init program. If you don't know what shells are available, `init=/bin/sh` should be a safe bet.
4. Once in the shell, we'll want to remount the root file system in read-write mode, instead of read-only mode (the default), then set the password:

```sh
# remount the root file system
mount -o remount,rw /
# set the root password
passwd
# sync cached writes to storage
sync
# unmount the root file system
umount /
```

5. Power off the computer
