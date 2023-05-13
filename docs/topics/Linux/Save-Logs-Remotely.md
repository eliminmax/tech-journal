# Linux: Save Logs remotely with rsyslog

Set up a server to collect logs from other servers

For this example, the server 10.0.4.12 is the logging server for the private networkk 10.0.4.0/22, and the web server at 10.0.4.4 sends its logs to it. I am basing this on a class lab in which the equivalent devices are both running CentOS 7

## Setup on Log Server

### Firewall Rules (firewalld)

```
[user@logserver01 ~]$ sudo firewall-cmd --permanent --add-port=514/tcp --add-port=514/udp
[user@logserver01 ~]$ sudo firewall-cmd --reload
```

### Configure rsyslog

edit the configuration file:
`[user@logserver01 ~]$ sudoedit /etc/rsyslog.conf`

Uncomment the following lines

```
$ModLoad imudp
$UDPServerRun 514

$ModLoad imtcp
$InputTCPServerRun 514
```
Restart the rsyslog service with `[user@logserver01 ~]$ sudo systemctl restart rsyslog`

## Setup on Log Client

Create an additional configuration file:
`[user@webserver01 ~]$ sudoedit /etc/rsyslog.d/send-to-log-server.conf`

Add the following line for UDP-based connections (faster, less reliable):
```
user.notice @10.0.4.12
```

Add the following line for TCP-based connections (slower, more reliable):
```
user.notice @@10.0.4.12
```

To send Authentication events specifically, you could instead set up the following:

```
auth,authpriv.* @10.0.4.12
```
## Save different logs to different files

Create a file in `/etc/rsyslog.d/` on the log server with the following contents to automatically save logs to subdirectories of `/var/log/remote-syslog` based on the hostname, date, and program name of the log source.

```conf
module(load="imudp")
input(type="imudp" port="514" ruleset="RemoteDevice")
template(name="DynFile" type="string"
        string="/var/log/remote-syslog/%HOSTNAME%/%$YEAR%.%$MONTH%.%$DAY%.%PROGRAMNAME%.log"
)
ruleset(name="RemoteDevice"){
        action(type="omfile" dynaFile="DynFile")
}
```
