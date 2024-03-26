<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Windows Admin Center

Download the installer from [the official site](https://www.microsoft.com/en-us/evalcenter/download-windows-admin-center) and run it.

When installing, do not check the WinRM over HTTPS only box - that broke things for me.

Under settings/extensions, install the Active Directory and DNS extensions.

To be able to manage a workstation, you'll need to launch PowerShell as admin and run `Enable-PSRemoting` on the workstation.
