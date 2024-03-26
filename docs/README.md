<!--
SPDX-FileCopyrightText: 2020-2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Eli Array Minkoff's Tech Notes

## Contents

<!-- vim-markdown-toc GitLab -->

* [How I maintain this site](#how-i-maintain-this-site)
  * [Software and Configuration](#software-and-configuration)
    * [MkDocs](#mkdocs)
    * [Git](#git)
    * [Neovim](#neovim)
      * [Plugins](#plugins)

<!-- vim-markdown-toc -->

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

MkDocs is installed into a `pipx`-managed venv, with the following extra packages injected, though not all are used:

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

I use [Neovim](https://neovim.io) to write the pages.

##### Plugins

I use the following plugins for Neovim (maintained with [vim-plug](https://github.com/junegunn/vim-plug))

* [airblade/vim-gitgutter](https://github.com/airblade/vim-gitgutter/) - git change information
* [mzlogin/vim-markdown-toc](https://github.com/mzlogin/vim-markdown-toc/) - generate table of contents
* [plasticboy/vim-markdown](https://github.com/plasticboy/vim-markdown/) - markdown-specific extras
  * [godlygeek/tabular](https://github.com/godlygeek/tabular/) - used by plasticboy/vim-markdown for some extra functionality
* [tpope/vim-fugitive](https://github.com/tpope/vim-fugitive/) - git integration that's "so good, it should be illegal!"
* [tpope/vim-commentary](https://github.com/tpope/vim-commentary/) - comment or uncomment lines in bulk
* [xero/sourcerer.vim](https://github.com/xero/sourcerer.vim/) - A dark color scheme that I find readable and easy on the eyes, particularly in darker environments.
* [preservim/nerdtree](https://github.com/preservim/nerdtree/) - File tree sidebar for vim
  * [Xuyuanp/nerdtree-git-plugin](https://github.com/Xuyuanp/nerdtree-git-plugin/) - git integration for NERDTree
  * [ryanoasis/vim-devicons](https://github.com/ryanoasis/vim-devicons/) - Uses "[Nerd Fonts](https://github.com/ryanoasis/nerd-fonts)" to provide pseudo-icons for files in NERDTree
  * [jcharum/vim-nerdtree-syntax-highlight](https://github.com/jcharum/vim-nerdtree-syntax-highlight/) - Color coding for files in NERDTree
* [mg979/vim-visual-multi](https://github.com/mg979/vim-visual-multi/) - Multi-cursor implementation
* [HiPhish/jinja.vim](https://github.com/HiPhish/jinja.vim/) - Syntax highlighting for Jinja2 template files

*(this is not an exhaustive list of plugins, and all of these are in use in maintaining this site. Really.)*

Additionally, I use an ever-changing list of syntax highlighting plugins.
