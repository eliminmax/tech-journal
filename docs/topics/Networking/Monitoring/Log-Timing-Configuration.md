# Logging Configuration on various systems

<!-- vim-markdown-toc GitLab -->

* [Linux/Unix](#linux-unix)
  * [Rsyslog](#rsyslog)
    * [Send Logs to Remote Logging Server](#send-logs-to-remote-logging-server)
    * [Make logs include UTC timestamp](#make-logs-include-utc-timestamp)

<!-- vim-markdown-toc -->

## Linux/Unix

### Rsyslog

#### Send Logs to Remote Logging Server

[Documented in Linux: Save Logs Remotely](../../Linux/Save-Logs-Remotely.md)

#### Make logs include UTC timestamp

1. Comment out the line "`$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat`" in the file `/etc/rsyslog.conf`
2. Restart `rsyslog.service`: `# systemctl restart rsyslog.service`
