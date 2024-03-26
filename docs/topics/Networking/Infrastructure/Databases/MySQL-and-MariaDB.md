<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# MySQL and MariaDB

{{ j2_template_note }}

<!-- vim-markdown-toc GitLab -->

* [Note](#note)
* [Installation and Setup](#installation-and-setup)
* [The SQL language](#the-sql-language)
  * [Create a User](#create-a-user)
  * [Create a Database](#create-a-database)
  * [Privilege Management](#privilege-management)
* [Galera Clustering](#galera-clustering)
  * [Setup](#setup)
    * [Installation:](#installation)
    * [Configuration](#configuration)
    * [Bootstrap](#bootstrap)

<!-- vim-markdown-toc -->

## Note

Use MariaDB. It's a fork of MySQL made by many of the original creators of MySQL, after it was bought out by Oracle. It's compatible with MySQL, and does not have the baggage of being owned by Oracle

## Installation and Setup

In general, you can install using a package manager. For instance, on Debian, you can run `apt install default-mysql-server default-mysql-client`. This ships the recommended MySQL/MariaDB server for the current release.

On Debian 11.6 (Bullseye), that is MariaDB 10.5. Once installed, run `mysql_secure_installation` as root to change some settings to more secure alternatives.

## The SQL language

Run `mysql -u root` (or `mysql -u root -p` if you set a password for the `'root'@'localhost` MySQL user). This will give you a command-line interface to run SQL commands.

SQL is a domain-specific language used for interacting with databases. Different database servers support different SQL dialects, but because MariaDB was forked from MySQL, they are almost fully compatible, though they are diverging over time. See [this page](https://mariadb.com/kb/en/mariadb-vs-mysql-compatibility/) on MariaDB's Knowledge Base for more information on the state of compatibility between them.

I am not particularly good with SQL, but I am able to do some basic things with it, including the following:

### Create a User

```sql
-- create a user called 'mediawiki', which can log in locally with password "SOME_PASSWORD"
CREATE USER 'mediawiki'@'localhost' IDENTIFIED BY "SOME_PASSWORD";
-- create a user called 'dbuser' which can log in from the 10.5.0.0-10.5.0.254 with no password
CREATE USER 'dbuser'@'10.5.0.0/255.255.255.0';
-- create a user called 'eliminmax' which can log in from any client host in the 192.0.2.0/24 TEST_NET with the password "████████████████████████████████"
CREATE USER 'eliminmax'@'192.0.2.%' IDENTIFIED BY "████████████████████████████████";
```

Note: `%` is a wildcard, and "`'user'@'%'`" and "`'user'`" are equivalent, allowing `user` to log in from any client host.

### Create a Database

```sql
CREATE DATABASE 'databaseName';
```

### Privilege Management

```sql
-- Allow unrestricted access to databaseName for user 'mediawiki'@'localhost'
GRANT ALL PRIVILEGES on 'databaseName' to 'mediawiki'@'localhost';
-- Apply privileges
FLUST PRIVILEGES;
```

For more fine-tuned privilege management, see [this page](https://mariadb.com/kb/en/grant/) on MariaDB's Knowledge Base.

## Galera Clustering

In larger-scale environments, it is common (and good practice) to run a dedicated database server, rather than running the database locally on the same system as the software that uses it. It is possible to set up a Galera cluster - a highly-available, redundant MySQL/MariaDB instance. It is important to have at least 3 nodes - in a 2-node cluster, if one goes down temporarily, when it comes back up, they can't determine which version of the database is the "real" one. If there are more than 2 hosts, it will defer to the other 2 if they are in sync.

### Setup

#### Installation:

* Ubuntu 22.04:
  * `apt install galera-4`

#### Configuration

In the MySQL configuration file/s, add a section with the following template:
{% raw %}
```conf
[galera]
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://{{ galera.cluster_addresses }}"
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
wsrep_cluster_name="{{ galera.cluster_name }}"
wsrep_cluster_name="{{ host.ip_address }}"
```
{% endraw %}
The `galera` configuration options should be the same for all nodes in the cluster, but the `host` settings should be per-host.

#### Bootstrap

You need to "bootstrap" a cluster, once you've set up the configuration

* If running `mysqld` manually, on the first node in the cluster, run `mysqld --wsrep-new-cluster`
* on systemd-based systems, run `galera_new_cluster`
* on SysVinit-based systems, run `service mysql bootstrap`
