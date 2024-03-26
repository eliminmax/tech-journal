<!--
SPDX-FileCopyrightText: 2023 - 2024 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# DNS Enumeration

Various techniques and tools for DNS enumeration:

## Find a DNS server

### nmap + awk

Print IP addresses of all servers with port 53 open on 192.0.2.0/24\*

```sh
sudo nmap -sU -sT -Pn -p53 --open 192.0.2.0/24 -oG - | awk '/Ports: 53\/o/{print $2}'
```

### ksh, bash, or yash + coreutils timeout

```sh
for i in 192.0.2.{1..254}; do do if timeout .1 bash -c "echo >/dev/tcp/$ip/53" 2>/dev/null; then echo $ip; fi;  done
```

```sh
for i in 192.0.2.{1..254}; do do if timeout .1 ksh -c "echo >/dev/tcp/$ip/53" 2>/dev/null; then echo $ip; fi;  done
```

```sh
for i in 192.0.2.{1..254}; do do if timeout .1 yash -c "echo >/dev/tcp/$ip/53" 2>/dev/null; then echo $ip; fi;  done
```

\* I chose this address range to avoid any real-world risk - it's TEST-NET-1, one of 3 /24 blocks explicitly reserved for use in documentation.

## Host discovery

Using a specified DNS server, scan all hosts in a /24 network:

* [Bash script](https://github.com/eliminmax/cncs-journal/blob/master/SEC335/week3/dns-resolver.sh)
* [Powershell script](https://github.com/eliminmax/cncs-journal/blob/master/SEC335/week3/dns-resolver.ps1)
