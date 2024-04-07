<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# SUID Binaries

Typically, a user runs a program on a Unix-like system, it runs with the privileges of the user. If the owner sets the special `SUID` permission bit, however, the program gains the owner's privileges.

If the owner happens to be the `root` user, then the program will run with root-level permissions.

Typically, programs like `su`, `sudo`, `pkexec`, which are used to run processes as another user, as well as `passwd` and `chsh`, which change root-only files, have the `SUID` bit set. They are specifically designed for that purpose, and that's not a problem.

Sometimes, sysadmins will set the `SUID` bit on programs that may not need it. For instance, `ping`  needs to be able to write raw network sockets, and historically, that required it to have the `SUID` bit, though on modern Linux systems, more fine-tuned approaches are used in that case.

In one case, the `beep` command, which causes the CPU's speaker to beep, had a buffer overflow bug, which, if it had the `SUID` bit set, could be used to gain root access. Why would the `SUID` bit ever be set on a program like that? Because it needs to be able to write to the PC speaker device file, and the docs encouraged setting the `SUID` bit. They don't anymore, and, in fact, on modern Debian systems, `beep` refuses to work if run with `sudo` or with the `SUID` bit set.

See the (hilarious) [website](https://holeybeep.ninja/) for more info on that mess.

Anyway, to find all SUID binaries on the root filesystem, run the following:

```sh
find / -type f -perm /u=s -print 2>/dev/null
```

To set the `SUID` bit, if you absolutely know what you are doing, you can set it with `chmod`:

```sh
chmod u+s /path/to/file
```
