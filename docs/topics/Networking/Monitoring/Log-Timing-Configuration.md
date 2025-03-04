<!--
SPDX-FileCopyrightText: 2022 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Logging Configuration on various systems

## Linux/Unix

### Rsyslog

#### Send Logs to Remote Logging Server

[Documented in Linux: Save Logs Remotely](../../Linux/Save-Logs-Remotely.md)

#### Make logs include UTC timestamp

1. Comment out the line "`$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat`" in the file `/etc/rsyslog.conf`
2. Restart `rsyslog.service`: `# systemctl restart rsyslog.service`
