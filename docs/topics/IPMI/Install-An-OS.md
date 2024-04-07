<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# SuperMicro Super Server IPMI: Install an OS

## From USB Installer

1. Navigate to the IP address of the IPMI interface, and enter the login credentials

2. Attach the installation drive

3. Navigate to **Remote Control** » **iKVM/HTML5**, then click the **iKVM/HTML5** button.

4. Click the button in the bottom of the display area to enable the virtual keyboard.

5. Open the **Power Control** menu and click **Set Power On** (or **Set Power Reset** if it's already on), then click into the screen within the display area

6. Once prompted, **repeatedly click** the F11 key ***on the virtual keyboard***.

7. In the boot menu, select the UEFI entry for the USB installer.

8. Install the OS, using the virtual keyboard for any keys that might have special meaning on the system you're working from (such as function keys).
