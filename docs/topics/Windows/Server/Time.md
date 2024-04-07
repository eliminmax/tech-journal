<!--
SPDX-FileCopyrightText: 2022 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Windows Server - Time

## Configure NTP in cmd.exe:

```
w32tm /config /syncfromflags:manual /manualpeerlist:pool.ntp.org
net stop w32time
net start w32time
w32tm /resync
w32tm /query /source
```
