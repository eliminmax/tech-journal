# Logging Configuration on various systems

## Linux/Unix

### Rsyslog

#### Send Logs to Remote Logging Server

[Documented in Linux: Save Logs Remotely](/topics/Linux/Save-Logs-Remotely)

#### Make logs include UTC timestamp

1. Comment out the line "`$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat`" in the file `/etc/rsyslog.conf`
2. Restart `rsyslog.service`: `# systemctl restart rsyslog.service`
