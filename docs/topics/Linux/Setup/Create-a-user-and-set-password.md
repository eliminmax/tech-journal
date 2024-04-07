<!--
SPDX-FileCopyrightText: 2020 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Linux User Setup

{{ j2_template_note }}

<!-- vim-markdown-toc GitLab -->

* [Create a User](#create-a-user)
* [Set Password](#set-password)

<!-- vim-markdown-toc -->

## Create a User
{% raw %}
run the following as root, replacing `{{ PLACEHOLDER_VALUES }}` as needed:

 `useradd -d {{ HOMEDIR }} -G {{ GROUPS }} -s {{ SHELL }} -m {{ USERNAME }}`

   * `{{ HOMEDIR }}`: path to the new user's home directory
   * `{{ GROUPS }}`: should be a comma separated list of groups
   * `{{ SHELL }}`: shell that the user defaults to on login. If you're not sure, go with `/bin/bash`
   * `{{ USERNAME }}` name of the new user

## Set Password

To set the password for the new user, use the `passwd` command.

`# passwd {{ USERNAME }}`

  * `{{ USERNAME }}`: name of the user who's password is changing

If you are setting/resetting someone else's password, pass the `-e` flag to make change the password when they log in

`passwd -e {{ USERNAME }}`
{% endraw %}
