<!--
SPDX-FileCopyrightText: 2022 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Install and configure Graylog

{{ j2_template_note }}

## RHEL 7/8, CentOS 7, AlmaLinux 8, Rocky Linux 8, etc.

*Installation instructions adapted from [Graylog CentOS Installation](https://docs.graylog.org/docs/centos) and [Graylog server.conf documentation](https://docs.graylog.org/docs/server-conf)*

### Manual Approach

#### Step 1 - Install Software

As root/with `sudo`, run the following:

```sh
# install openjdk
yum install -y java-11-openjdk-headless

# enable epel-release repo
yum install -y epel-release

# install pwgen
yum install -y pwgen

# install policycoreutils-python
yum install -y policycoreutils-python

# add mongodb repo
cat >/etc/yum.repos.d/mongodb-org.repo <<EOF
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/\$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
EOF 

# add elasticsearch repo key
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

# add elasticsearch repo
cat >/etc/yum.repos.d/elasticsearch.repo <<EOF
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/oss-7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF

# add graylog repo
rpm -Uvh https://packages.graylog2.org/repo/packages/graylog-4.2-repository_latest.rpm

# install mongodb
yum install -y mongodb-org

# install elasticsearch
yum install -y elasticsearch-oss

# install graylog
yum install -y graylog-server graylog-{enterprise,integrations,enterprise-integrations}-plugins
```

#### Step 2 - Configure and Start Services

As root/with `sudo`, run the following:
{% raw %}
```sh
# Configure elasticsearch
cat >>/etc/elasticsearch/elasticsearch.yml <<EOF
cluster.name: graylog
action.auto_create_index: false
EOF

# Configure graylog
# set admin password
sed -i "s/root_password_sha2.*$/root_password_sha2 = {{ ADMIN_PASSWORD }}/g" /etc/graylog/server/server.conf

# set password_secret
sed -i "s/password_secret.*$/password_secret = $(pwgen -N 1 -s 96 | tr -d '\n')/g" /etc/graylog/server/server.conf

# set bind address (format: 198.0.2.176:9000 or for.example:8080)
sed -i "s/#http_bind_addresss.*$/http_bind_addresss = {{ HTTP_BIND }}/g"

# Reload SystemD Daemons
systemctl daemon-reload

# Start MongoDB
systemctl enable --now mongod.service

# verify MongoDB is running
systemctl --type service --state active | grep mongod

# Start elasticsearch
systemctl enable --now elasticsearch

# verify elasticsearch is running
systemctl --type service --state active | grep elasticsearch

# start graylog
systemctl enable --now graylog-server

# verify graylog is running
systemctl --type service --state active | grep graylog
```{% endraw %}

#### Step 3 - Firewall and SELinux Stuff

```sh
# configure SELinux and firewall
setsebool -P httpd_can_network_connect 1
semanage port -a -t http_port_t -p tcp 9000
semanage port -a -t http_port_t -p tcp 9200
semanage port -a -t mongod_port_t -p tcp 27017
firewall-cmd --add-port=9000/tcp --permanent
firewall-cmd --reload
```

### Automated approach to Steps 1, 2, and 3

Download my setup script - it just does all of the above in one interactive script

```sh
# Download script
wget https://raw.githubusercontent.com/eliminmax/cncs-journal/master/SEC350/graylog-deploy-el.sh

# Read over script to ensure you trust it
less graylog-deploy-el.sh

# run script
chmod +x graylog-deploy-el.sh
sudo ./graylog-deploy-el.sh
```

#### Step 4 - Rsyslog Replacement

Open up the web interface, and log in with the username `admin` and the password set in step 2.
{% raw %}
Navigate to the **System > Inputs** menu, and, in the **Select Input** Syslog UDP (the last option). Configure the options you want, and make sure that the port you select is not already in use (e.g. by the `rsyslog` service). Also, make sure to add the port you select through the firewall (`sudo firewall-cmd --permanent --add-port={{ port }}/udp && sudo firewall-cmd --reload`) (I forgot to do the latter when setting this up for a class lab.){% endraw %}

#### Step 5 - change rsyslog client settings

On the rsyslog client - that is, the server which sends its log messages to Graylog, add or change the rules to point to the address and port configured in Step 4, then restart the rsyslog service.

**Example - modify the Log Client set up in [Linux: Save Logs Remotely](../../Linux/Save-Logs-Remotely.md#setup-on-log-client)**

Assuming the rsyslog input is running on port 1514, edit /etc/rsyslog.d/send-to-log-server.conf, so that it reads as follows:

```
user.notice @10.0.4.12:1514
```

# Send Windows EventLogs to Graylog with Sidecar and Winlogbeat

0. On the Graylog web console, create a new input of the type **Beats**

1. Navigate to **System/Sidecars**, and generate an API token. Make sure not to lose it, and keep the Graylog web console open here

2. On the Windows system, download and install Graylog Sidecar, Download the latest Graylog Sidecar installer from [GitHub releases](https://github.com/Graylog2/collector-sidecar/releases/latest) to its own directory - e.g. `C:\Program Files\Graylog\Sidecar`

3. At the command line, run the following:
{% raw %}
```
.\graylog_installer_X.Y.Z-a.exe /S -SERVERURL=http://{{ logserver }} :{{ port }}/api -APITOKEN={{ the_aforementioned_token }}
.\graylog_installer_X.Y.Z-a.exe -service install
.\graylog_installer_X.Y.Z-a.exe -service start
```{% endraw %}

**At this point, Graylog should detect the sidecar. If it doesn't, fix that before moving on**

4. Back on Graylog, create a collector configuration, adjust it, and apply it to the collector you've set up, then start it.
