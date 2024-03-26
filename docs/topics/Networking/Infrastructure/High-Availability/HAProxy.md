<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# HAProxy

## Installation

Most Linux distros have HAProxy in their official repositories. To determine the package name for a given distro, I would use [Repology](https://repology.org/project/haproxy/versions).

To use HAProxy, you need to set up a configuration file. On Debian and Ubuntu systems, the format for this file is in the compressed text document `/usr/share/doc/haproxy/configuration.txt.gz`, and a simple introduction which is good enough for basic use cases can be found [at this page](https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers/) on haproxy.com.

### Non-HTTP Apps

#### MySQL/MariaDB

For my [Redundancy/High Availablity project](https://github.com/eliminmax/cncs-journal/wiki/Working-Notes%3A-SEC440%3A-High-Availability-Project), I had to use HAProxy to proxy requests between 2 Nextcloud servers and a Galera cluster (i.e. a redundant, distributed MySQL/MariaDB database). There were a few extra steps to this - I had to create `haproxy@10.0.6.11` and `haproxy@10.0.6.12` users on the Galera cluster, which allowed `haproxy` to determine which of the servers were alive at any given time. I added `nextcloud-user` database users to the cluster, with the same password and permissions as those on the nextcloud servers themselves, to allow HAProxy to pass on the requests. I then added the following to the `ha1` server's HAProxy configuration, and another entry that was identical save for the name to the `ha2` server's configuration.

```conf
listen ha1_sql
	bind 10.0.6.10:3306
	balance roundrobin
	mode tcp
	option tcpka
	option mysql-check user haproxy
	server u1 10.0.6.21:3306 check weight 1
	server u2 10.0.6.22:3306 check weight 1
	server u3 10.0.6.23:3306 check weight 1
```
