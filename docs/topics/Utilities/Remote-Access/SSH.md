<!--
SPDX-FileCopyrightText: 2021 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# SSH

This is a page documenting the use of **OpenSSH**, the *de facto* standard system for remote access to Linux/Unix servers.

## Security

### Block Root Login

> Prevent anyone from logging into root using ssh

0. open */etc/ssh/sshd_config*

1. search for the entry `PermitRootLogin`

   * In **Vi**/**Vim**, in Normal mode, type `/PermitRootLogin` to search for it, and type `n` and `N` to cycle through all occurrences.

   * If you can't find the entry, add it.

2. If it's commented, uncomment it. Set the value to `no`.

   * The entry should now read `PermitRootLogin no`.

3. Restart the OpenSSH Daemon: `# systemctl restart sshd`.

### Set Up Key-based Authentication

> Generate a public/private key pair, and replace password-based authentication with something more secure

#### With OpenSSH Client

> Available in **PowerShell** for Windows, and built in to Unix/Linux systems (including macOS, using **Terminal**)

0. Run `$ ssh-keygen`, and answer the prompts it provides.

1. Append the public key (*~/.ssh/id_rsa.pub by default*) to the server's *~/.ssh/authorized_keys* file.

* on Unix and Linux, this can be done automatically with `ssh-copy-id user@host`

#### With PuTTY

> Available on Windows

0. Open **PuTTYgen**

   * ***Start -> All Programs -> PuTTY -> PuTTYgen*** 
1. Select a key type (2048-bit RSA should be fine) and move the mouse around the window.

2. (Optional, but recommended): Specify a passphrase

3. Save both the private and public keys.

4. Append the **public** key to the end of *~/.ssh/authorized_keys* file on whatever servers you want to use it with.

**IMPORTANT**: You should ***NEVER, EVER*** publish or share the private key (*~/.ssh/id_rsa* by default). It is called the private key for a reason - treat it with at least as much care as a password. 

### How to use SSH-Agent

> If you are password-protecting your private key, you can set up **SSH-Agent** to remember the password, so you don't need to type it every time you run an ssh command. That said, you should set a reasonably short lifetime.

0. run `$ eval $(ssh-agent -s)`

   * on Windows, run `> ssh-agent`

   * this will output the PID of **ssh-agent**. You might want to `kill` the process when you're done using **SSH**.

1. run `$ ssh-add -t 3600`

   * `-t 3600` sets the lifetime to one hour (3600 being 1 hour in seconds). You can specify whatever number you prefer.

## SCP

### Copy files to/from machines with SCP

> Copy files and folders too and from other machines using **scp**

For individual files, run `$ scp source destination`/

For directories, run `$ scp -r source destination`, to copy directory contents recursively.

To specify a remote path, "*user@host:/path/to/filename.ext*"

You can also use pattern matching:

   `$ scp eli@203.0.113.233:~/Pictures/*.png .` will copy any files with the `.png` extension in the *~/Pictures* directory on the remote server to the current directory.

### Dealing with known_hosts file

> When attempting to connect to a new system with the same hostname or IP address as a system you previously connected to, the OpenSSH client will refuse to connect. You can get around that by editing the known_hosts file (*~/.ssh/known_hosts)

#### Option 1: Delete the offending line

Find the offending line in the known_hosts file, and delete it

#### Option 2: Empty or delete the known_hosts file

Delete the file, or clear its contents. A bit drastic, but it's an option.

### Edit remote files with Vim

> If you want to edit a remote document with a local copy of **Vim**, you can do so with the following formats:

`scp://user@host//absolute/path/to/file.ext`

`scp://user@host/path/from/home/to/file.ext`

Example 1: open user@host's *~/Documents/TODO.txt* from command line: `$ vim scp://user@host/Documents/TODO.txt`

Example 2: open user@hosts's */var/www/html/about/index.html* from within vim:

   * in normal mode, type `:e scp://user@host//var/www/html/about/index.html`
