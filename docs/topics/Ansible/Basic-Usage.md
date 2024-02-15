# Ansible: Basic Usage

{{ j2_template_note }}

<!-- vim-markdown-toc GitLab -->

* [Basic Usage](#basic-usage)
  * [Set up an inventory file](#set-up-an-inventory-file)
  * [Run a command on all hosts in inventory](#run-a-command-on-all-hosts-in-inventory)
  * [Run a module on all hosts in inventory](#run-a-module-on-all-hosts-in-inventory)
  * [Group Headers](#group-headers)
* [Playbooks](#playbooks)
  * [Running an Ansible Playbook](#running-an-ansible-playbook)
  * [Templating](#templating)
  * [Conditionals](#conditionals)
* [Galaxy](#galaxy)
  * [Installing Roles](#installing-roles)

<!-- vim-markdown-toc -->

## Basic Usage

### Set up an inventory file

At its most basic, an inventory file contains a list of hostnames, with one per line. If no inventory file is specified, **Ansible** defaults to */etc/ansible/hosts*

You can also create host groups by listing them under a group name specified within square brackets.

You can also set per-host or per-group variables that can be referenced in Jinja2 templates (see [below](#templates)). Some variables, like `ansible_user`, are used to change ansible's default behavior. In the following example, Ansible will attempt to connect to servers listed under `[groupa]` as `ansible_setup_user`, while it will attempt to connect to servers listed under `[groupb]` as `administator`

```conf
[groupa]
host1.a.example.com color=red location=philadelphia
host2.a.example.com color=green location=burlington
[groupb]
host1.b.example.com color=blue location=toronto

[groupa:vars]
ansible_user=ansible_setup_user

[groupb:vars]
ansible_user=administator
```

### Run a command on all hosts in inventory
{% raw %}
```sh
ansible all -a {{ COMMAND }} -i {{ INVENTORY_FILE }}
```
{% endraw %}
### Run a module on all hosts in inventory
{% raw %}
```sh
ansible all -m {{ MODULE }} -i {{ INVENTORY_FILE }}
```
{% endraw %}

### Group Headers

You can group hosts with **Group Headers**, and specifically target them by **Header**:

In *inventory.txt*:

```plaintext
[grouped-hosts]
grouped-host-0
grouped-host-1
```
Ping tagged machines:
{% raw %}
`ansible grouped-hosts -m {{ MODULE }} -i {{ INVENTORY_FILE }}`
{% endraw %}
## Playbooks

Playbooks are YAML files that define a more complex processes than running individual commands or modules.Since this is just a syntax and filetype-detection plugin there is nothing to configure, once a file has been identified as a Jinja file it will be highlighted appropriately. Any file with the extension .jinja will be recognised as a Jinja file.

Here is an example playbook, which ensures that **Apache** is up-to-date and running on a RHEL/CentOS system (copied from [middlewareinventory.com](https://www.middlewareinventory.com/blog/ansible-playbook-example/#Ansible_Playbook_Example))

```yaml
---
  - name: Playbook
    hosts: webservers
    become: yes
    become_user: root
    tasks:
      - name: ensure apache is at the latest version
        yum:
          name: httpd
          state: latest
      - name: ensure apache is running
        service:
          name: httpd
          state: started
```

### Running an Ansible Playbook
{% raw %}
```sh
ansible-playbook -i {{ INVENTORY_FILE }} {{ PLAYBOOK_FILE }}
```
{% endraw %}
### Templating

Ansible playbooks can use Jinja2 templates to dynamically generate configuration files they reference, or even parts of themselves.

For instance, I have a simple playbook I use to determine how a given Linux host's distro is identified by Ansible, helping me figure out how to modify behavior for different distros. It has the following contents
{% raw %}
```yaml
---
- name: Get Server Distribution
  hosts: all
  gather_facts: true
  tasks:
    - name: Ansible fact OS
      debug:
        msg: "Distro: {{ ansible_distribution }} (like {{ ansible_distribution_file_variety }}); Distro Family: {{ ansible_os_family }}"
```
{% endraw %}
See [the official docs](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html) for details.

### Conditionals

Conditional execution of a task can help create a more portable playbook. For example, I have a playbook called `guest-init.playbook.yml` that I use when setting up new Linux VMs. The following is an excerpt of that file that illustrates the usage of conditionals

```yaml
    - name: Update apt cache on Debian-like systems
      ansible.builtin.apt:
        update_cache: true
      when: ansible_facts['os_family'] == "Debian"

    - name: Enable the Extra Packages for Enterprise Linux repository on RHEL-like systems
      ansible.builtin.dnf:
        name: epel-release
        state: latest
      when: ansible_facts['os_family'] == "RedHat" and ansible_facts['distribution'] != "Fedora"

    - name: Upgrade installed software (sensible distros)
      ansible.builtin.package:
        name: '*'
        state: latest
      when: ansible_facts['os_family'] != "Archlinux"

    - name: Upgrade installed software btw
      community.general.pacman:
        update_cache: yes
        upgrade: yes
      when: ansible_facts['os_family'] == 'Archlinux'
```

With the conditionals, I have managed to do the following:
* On Debian, Ubuntu, and related systems, update the apt cache.
* On RedHat-like systems, other than Fedora, enable EPEL
* On non-Arch systems, update installed packages
* On Arch systems, update installed packages. This is done separately because `ansible.builtin.package` does not work with Arch's `pacman` package manager.

## Galaxy

Quoting the [Galaxy Documentation homepage](https://galaxy.ansible.com/docs/) (and actually using block quotes as intended for once):

> ### About Galaxy
>
> Galaxy is a hub for finding and sharing Ansible content.
>
> Use Galaxy to jump-start your automation project with great content from the Ansible community. Galaxy provides pre-packaged units of work known to Ansible as Roles, and new in Galaxy 3.2, Collections.

### Installing Roles

Default path (requires root)
{% raw %}
```sh
ansible-galaxy install {{ role_id }}
```

Alternate path (does not require root)

```sh
ansible-galaxy install --roles-path {{ install_path }} {{ role_id }}{% endraw %}
```
