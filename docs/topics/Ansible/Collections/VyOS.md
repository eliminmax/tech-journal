<!--
SPDX-FileCopyrightText: 2023 - 2024 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Ansible: VyOS

<!-- vim-markdown-toc GitLab -->

* [Line by Line configuration](#line-by-line-configuration)

<!-- vim-markdown-toc -->

Ansible has the ability to simplify the configuration of VyOS systems, using a few different techniques.

If you have the `Vyos.Vyos` collection installed, you can specify a section within a task to configure a server.

Note that you must set the `ansible_connection` variable to `network_cli` for this to work.

## Line by Line configuration

Instead of running the following:

```
configure
set system host-name vyos.for.example
```

You can add the following to an ansible playbook

```yaml
- name: set hostname
  vyos.vyos.vyos_config:
    lines:
      - set system host-name vyos.for.example
```

By specifying the desired hostname in a variable, defined per-host, you can use one task on any number of hosts at once:
{% raw %}
```yaml
- name: set hostname
  vyos.vyos.vyos_config:
    lines:
      - set system host-name {{ hostname }}
```
{% endraw %}
You can also use a configuration file template - for class, I modified [this template](https://github.com/gmcyber/480share/blob/master/config.boot.j2) made by my professor.

```yaml
- name: load configuration template
  vyos.vyos.vyos_config:
    src: config.boot.j2
```
