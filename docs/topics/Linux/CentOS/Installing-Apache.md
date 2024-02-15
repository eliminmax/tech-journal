# CentOS: Installing Apache

To set up Apache, follow these simple steps (all commands require root privileges):

0. Install the Apache software by running the following command, and confirm installation: `yum install httpd`

1. Start the httpd service, and set it to run at boot: `systemctl start httpd && systemctl enable httpd`

2. Allow the http and https services through the firewall: `firewall-cmd --add-service=http --add-service=https --permanent && firewall-cmd --reload`
