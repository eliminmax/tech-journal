# Set Up AD DS and DNS

> This is made for Windows Server 2019 Standard Edition version 1809, and I can not guarantee it works on any other version of Windows Server.

## Setting Up Active Directory Domain Services for a domain

0. Open **Server Manager**.

1. Within the ***Manage*** menu, click ***Add Roles And Features***

2. Select the ***Role-based or feature-based installation***, then choose the ***Active Directory Domain Services*** option

3. When the installation completes, open the notification menu, and click ***Promote this server to a domain controller***

4. Select ***Add a new forest***, and enter the domain name

5. Set a DSRM password

6. Finish configuration and reboot

## Add a DNS record

0. Within **Server Manager**, switch to the ***DNS*** tab, right click the server, and click ***DNS Manager***

1. Expand the sidebar objects until you find the *Forward Lookup Zones* directory, and right click your domain within it. Click ***New Host (A or AAAA)...***.

2. Type in the hostname and IP in the proper fields, and check ***Create associated pointer (PTR) record***. Ignore the "DNS" Warning.

3. Right click the *Reverse Lookup Zones* directory and select ***New Zone...***, and configure it with the Network ID.

4. Open the configuration for your Forward Lookup Zone host from steps 1-2, uncheck ***Updated associated pointer (PTR) record***, click ***Apply***, then check it again, then click ***OK***.

5. On the device you want to add, change the DNS Server to the IP address of the server you've been working on.

   > **Note:** everything beyond this point assumes that you have at least one domain admin on the network. If you don't, before you continue, follow the instructions below to add a user, and add it to the group "Domain Admins"

6. On the device you want to add, open **Control Panel**, and navigate to *Control Panel\System and Security\System*, and open System Properties. 

7. Click the ***Change...*** button, and within the ***Member of*** box, select ***domain***, and enter the name of the domain. Enter a Domain Admin's username and password, then reboot.

## Add objects to the network

> This includes adding users

0. Within **Server Manager**, switch to the ***AD DS*** tab, and right click the server

1. Click ***Active Directory Users and Computers***

2. Find the desired location, right click it, and select ***New > \<desired object type>***

3. Enter the object name, and select the desired settings

## Add a user to a group

0. Within **Server Manager**, switch to the ***AD DS*** tab, and right click the server

1. Click ***Active Directory Users and Computers***

2. Navigate to the user, right click it, and select ***Add to a group***

3. Add the group's name to the ***Object Names*** list, and click ***OK***
