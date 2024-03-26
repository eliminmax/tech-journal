<!--
SPDX-FileCopyrightText: 2021 - 2024 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Linux: Join Active Directory

All commands on this page must be run with root-level privileges.

0. install the required software:

 * CentOS 7: `yum install realmd samba samba-common oddjob oddjob-mkhomedir sssd`
 * CentOS 8: `yum install realmd sssd oddjob oddjob-mkhomedir adcli samba-common samba-common-tools krb5-workstation authselect-compat`
 * Ubuntu  20.04: `apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit`

1. Join the domain with the following command `realm join --user=<a_domain_admin>@<domain> <domain>` (You will need to enter the domain admin's password)

    * If you run into the error `Could not get kerebos ticket: KDC reply did not match expectations`, you might have to retype the domain in "`<a_domain_admin>@<domain>`" in all caps. (On CentOS 8 and Ubuntu, I was able to join with the option `--user=eli.minkoff-adm@ELI.LOCAL`, but not with the option `--user=eli.minkoff-adm@eli.local`)

It's that simple. You can now log in with Active Directory accounts, and this system should now show up under the **Computers** directory in Active Directory Users and Computers
