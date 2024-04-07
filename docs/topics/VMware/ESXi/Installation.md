<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# ESXi: Installation and Initial Setup

## Installation

1. Boot up to the VMware ESXi installer. Once you see the box that reads `Welcome to the VMware ESXi...`, press enter.

2. Press F11 to agree to the EULA

3. Select the device to install it on, and press Enter. Continue to follow the prompts at the bottom of the box, in order to confirm the disk selection, set up the keyboard layout, set the root password **and ensure you don't lose it**, and dismiss the `CPU_SUPPORT WARNING` should it come up. *Note that in a production environment, the warning is important to make a not of.*

4. Press F11 to install

5. Reboot

6. Once it's back online, press F2 to do the post-installation setup

## Initial Setup

1. Ensure the proper ethernet interface is connected - you may need to add another cable if one of the interfaces is used to access something like IPMI.

2. Log in with the root credentials set during the installation, and navigate to the `Configure Management Network` menu entry. Go down the list of network devices, and use the space bar to toggle them as needed, such that only the `Connected` devices have the box checked. If needed, you can also configure IPv4 and DNS search domains in this menu. Once done, press "y" when prompted to restart the management network.
