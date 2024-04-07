<!--
SPDX-FileCopyrightText: 2020 - 2023 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Creating and Configuring Group Policy Objects

> This is made for Windows Server 2019 Standard Edition version 1809, and I can not guarantee it works on any other version of Windows Server.

## Create a Group Policy Object

1. Within **Server Manager**, click ***Tools*** on the menu bar, and select ***Group Policy Management*** from the drop-down

2. Right click the desired location, and click ***Create a GPO in this domain, and Link it here...***. (The actual location will be in *Domains\<domain>\Group Policy Objects\*, but this will add a link in the desired location)

3. Under the ***Scope*** tab, add any groups, users, and/or computers that the GPO should apply to

4. ***This step is only needed if you want to change permissions to view or edit this GPO.*** Under the ***Delegation*** tab, click ***Advanced...***, and set the permissions

**IMPORTANT:** Changes to User Configuration take effect at login, so any group members who are currently logged in will not see any effects until they log out and log back in. Computer Configuration changes take effect at boot, so any computers in the group will need to be rebooted for changes to take effect.

## Configure a Group Policy Object

1. Within **Server Manager**, click ***Tools*** one the menu br, and select ***Group Policy Mangement*** from the drop-down 

2. Along the sidebar, right click the GPO you want to edit, and select ***Edit...***

3. Configure the policies as desired

### Specific Use Cases

#### Map a Network Share to a Letter Drive

1. In the sidebar, navigate to `User Configuration > Preferences > Windows Settings`, and right click `Drive Maps`, and select **`New > Mapped Drive`**

2. In the *Location* field, enter the **Remote Path**, and select the desired drive letter.
 
3. Switch to the **`Common`** tab, and check the desired options.

4. To limit it to specific users, devices, groups, etc., click the `Targeting...` button, and create a **New Item** with the desired criteria.

5. Apply the settings.
