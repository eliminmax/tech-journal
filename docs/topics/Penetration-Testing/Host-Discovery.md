<!--
SPDX-FileCopyrightText: 2022 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Host Discovery

{{ j2_template_note }}

<!-- vim-markdown-toc GitLab -->

* [Methods](#methods)
  * [`ping`](#ping)
  * [`fping`](#fping)
  * [`nmap`](#nmap)

<!-- vim-markdown-toc -->

## Methods

### `ping`

Ping sends ICMP ECHO_REQUEST packets to a specified host, and note the replies

**Useful Flags**
{% raw %}
* `-c {{ number }}`: send `{{ number }}` ping requests - by default, it keeps sending them, not stopping until the process is killed.
* `-i {{ seconds }}`: wait `{{ seconds }}` between pings (default is 1)
{% endraw %}
To ping all hosts in the `192.0.2.0/24` (`TEST-NET-1`) IP subrange, and **only** export the IP addresses that reply, run the following in `bash`:

```sh
for ip in 192.0.2.{1..254}; do ping -c1 "$ip" &>/dev/null && echo "$ip"; done
```

<details>
<summary>Explanation</summary>
<code>192.0.2.{1..254}</code>: expands to <code>192.0.2.1 192.0.2.2</code>... <code>192.0.2.254</code>. The network and broadcast addresses are left out.<br>
...<code>&>/dev/null && echo "$ip"</code>: redirect stdout and stderr of the `ping` command to /dev/null, effectively discarding it. If the ping was successful, echo the IP address itself.
</details>

### `fping`

Like ping, it sends ICMP ECHO_REQUEST packets, but it handles multiple target hosts more elegantly

**Useful Flags**
{% raw %}
* `-a`: list all pinged hosts that are online
* `-g {{ network }}/{{ mask }}`: generate a list of target hosts from a network address and netmask
* `-g {{ start }} {{ stop }}`: generate a list of target hosts from the first and last host in a sequence
{% endraw %}
To do the same thing as above, and list all addresses successfully pinged in `TEST-NET-1`, using `fping` this time, it is far simpler than with the classic `ping`.

```sh
# option 1
fping -a -g 192.0.2.0/24 2>/dev/null
# option 2
fping -a -g 192.0.2.1 192.0.2.254 2>/dev/null
```

### `nmap`

This one's much more aggressive. Using it without permission from the owners of the target systems can get you in trouble. It's much more powerful, but much more complicated.

**Useful Flags**

* `-sn`: disable port scan - don't scan for open ports, only list hosts that are up
   * despite being a "ping scan", this does not actually limit itself to ICMP ECHO_REQUEST packets, also sending a TCP SYN to port 443, a TCP ACK to port 80, and a ICMP timestamp requests. *If running as a non-privileged *nix user, it skips the ICMP packets entirely.*
{% raw %}
* `-oG {{ file }}`: write output to `{{ file }}`, in a "greppable" (i.e. easy to programatically parse) format
   * (special case) `-oG -`: write output to stdout in a "greppable" format
{% endraw %}
Using the same ping sweep example as earlier, one approach could be as follows:

```sh
nmap -sn 192.0.2.1-254 -oG - | awk '/^Host: .*Status: Up$/ {print $2}'
```
