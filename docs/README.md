<!--
SPDX-FileCopyrightText: 2020 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Eli Array Minkoff's Tech Notes

## Contents

As a Computer Networking and Cybersecurity student at Champlain College, I had to maintain a tech journal in a GitHub wiki, containing both notes on the technologies used and reflections on particular assignments and labs. I no longer want to maintain that wiki, now that my studies are complete, but I found the process of working on technical documentation to be helpful in reinforcing the concepts as I learned them. I am migrating the technical documentation to this self-hosted site, and I intend to continue working on it. The original `cncs-journal` git repository and wiki are archived, but can still be viewed [here](https://github.com/eliminmax/cncs-journal/wiki).

## How I maintain this site

I write the documentation in Markdown, and then generate the site contents using [MkDocs](https://www.mkdocs.org/).

I maintain it `array-lenny`, a midrange Lenovo-era Thinkpad running Debian GNU/Linux Bookworm.

### Software and Configuration

#### MkDocs

As mentioned above, I use [MkDocs](https://www.mkdocs.org) to generate the site contents from the original Markdown.

The `MkDocs` configuration file ('`mkdocs.yml`') contents are as follows:

```yaml
{% include 'mkdocs.yml' %}
```

MkDocs is installed into a `pipx`-managed venv, with the following extra packages injected:

* `mkdocs-material`
* `markdown-callouts`
* `mdx-truly-sane-lists`
* `mkdocs-autorefs`
* `mkdocs-include-markdown-plugin`
* `mkdocs-macros-plugin`
* `mkdocs-redirects`
* `pymdown-extensions`

#### Git

I use Git for version control for the original Markdown.

#### Neovim

I use [Neovim](https://neovim.io) to write the pages. It has a config that is partially shared with Vim in `.vimrc`, and partially Neovim-specific in `~/.config/init.lua`. Both parts are in [my dotfiles repo](https://github.com/eliminmax/dotfiles).
