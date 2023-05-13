# Change Account Username on Linux

{{ j2_template_note }}

I have, on occasion, wanted to change the username of a pre-configured account on a Linux system. For example, when I first wrote the original iteration of this page, I had to set up a Red Hat Enterprise Linux instance on EC2 for [a class lab](https://github.com/eliminmax/cncs-journal/wiki/Working-Notes%3A-SYS265%3A-Amazon-EC2-Lab). The default username was `ec2-user`, but I prefer the username `eliminmax`, so I changed it, and created this page to document how I did it.

## Preparation

You must have the ability to run commands with `root`-level permissions without logging in as the user you want to rename. See the end of this document for a method if you want to rename your only admin user

Ensure that the user you want to rename has no currently-running processes. The following command lists running processes for the user, and should not have any output:

```sh
ps -u {{ old_username }} --no-headers
```

## Renaming the User

Run the following commands to rename the user, the user's home directory, and the user's primary group, which is typically named after that user.

```sh
# Rename the user itself
usermod -l {{ new_username }} {{ old_username }}
# Rename the user's home directory
usermod -d /home/{{ new_username }} -m {{ new_username }}
# Rename the user's primary group
groupmod -n {{ new_username }} {{ old_username }}
```

---

**References**

* [Linux userdel command and examples | Computer Hope](https://www.computerhope.com/unix/userdel.htm)
* [How to rename user in Linux (also rename group & home directory) | LinuxTechLab](https://linuxtechlab.com/rename-user-in-linux-rename-home-directory/)
* [How To Find Currently Logged In Users In Linux | OSTechNix](https://ostechnix.com/how-to-find-currently-logged-in-users-in-linux/)
<!--- vim: set ft=markdown.jinja :-->
