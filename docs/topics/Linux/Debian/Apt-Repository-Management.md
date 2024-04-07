<!--
SPDX-FileCopyrightText: 2023 - 2024 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# APT Repository Management

<!-- vim-markdown-toc GitLab -->

* [Notes on Terminology](#notes-on-terminology)
* [Adding repositories](#adding-repositories)
  * [Third-Party Repositories](#third-party-repositories)
    * [Extrepo](#extrepo)
      * [Non-Free Software](#non-free-software)
      * [Example - Installing Visual Studio Code](#example---installing-visual-studio-code)
    * [Manually Adding a Repository](#manually-adding-a-repository)
      * [Acquiring GPG key](#acquiring-gpg-key)
      * [Creating the Repository Configuration File](#creating-the-repository-configuration-file)
        * [Template for .list format](#template-for-list-format)
        * [Template for .sources format](#template-for-sources-format)
      * [Example: Installing PowerShell on Debian Bullseye](#example-installing-powershell-on-debian-bullseye)

<!-- vim-markdown-toc -->

## Notes on Terminology

On this page, I use the terminology preferred by the Debian project, but some terms might be confusing, so I'm going to briefly define them.

* "non-free": software that violates the Debian Free Software Guidelines - in other words, proprietary
* "contrib": software packages that do not contain non-free components, but depend on external non-free components.

## Adding repositories

Typically, to add a repository, you need to add a configuration file and a GPG key. For the latter, online tutorials often instruct users to use the deprecated and insecure `apt-key` tool, which is not a good idea, for several reasons. The main issue is that the key is added to the **/etc/apt/trusted.gpg.d** directory, which allows them to sign for packages on any repository. Following the [princibal of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege), it would be better to only allow it to sign for packages on the specific repositories that need it. There are a few ways to go about this, detailed below.

Note that almost all commands on this page require root-level permissions to run, so unless otherwise noted, assume they need to be run as root. This can be done with `sudo`.

### Third-Party Repositories

#### Extrepo

The preferred way to add third-party repositories for Debian systems, [extrepo](https://salsa.debian.org/extrepo-team/extrepo)m consists of a curated list of external repositories and their associated GPG keys, as well as a command-line tool to interact with that list. There are four `extrepo` subcommands:
* `extrepo search <pattern>` finds all repositories that match a search for `<pattern>`. Omitting a pattern lists all repositories available to `extrepo`. This does not require root privileges
* `extrepo enable <repo>` enables the repository `<repo>`
* `extrepo disable <repo>` disables the repository `<repo>`
* `extrepo update <repo>` updates the local **.sources** file in **/etc/apt/sources.list.d** for `<repo>` to match the latest metadata.

##### Non-Free Software

Much of the software available in `extrepo`-managed repositories is considered to be "non-free" or "contrib" (see [above](#notes-on-terminology)), and `extrepo` will refuse to install such software by default. If you want to install non-free or contrib software, you can edit the file at **/etc/extrepo/config.yaml**, and add `contrib` and `non-free` to the `enabled_policies` section. On a Debian Bullseye system where that has been done, the contents of that file are as follows:

```yaml
---
url: http://extrepo-team.pages.debian.net/extrepo-data
dist: debian
version: bullseye
# To enable repositories that host software with non-DFSG-free licenses,
# uncomment "contrib" and/or "non-free" in the list below.
enabled_policies:
- main
- contrib
- non-free
```

##### Example - Installing Visual Studio Code

1. Check if available through extrepo
  * running `extrepo search vscode` returns the following:
```yaml


Found vscode:
---
description: Microsoft Visual Studio Code repo
gpg-key-checksum:
  sha256: 2cfd20a306b2fa5e25522d78f2ef50a1f429d35fd30bd983e2ebffc2b80944fa
gpg-key-file: vscode.asc
policy: non-free
source:
  Architectures: amd64
  Components: main
  Suites: stable
  Types: deb
  URIs: https://packages.microsoft.com/repos/vscode


```

2. Because the `policy` is `non-free`, you'll need to have added `non-free` to `enabled_polices` in **/etc/extrepo/config.yaml**. If you have not done so, you can run the following:

```sh
sed -i '/- non-free/s/^# //' /etc/extrepo/config.yaml
```

<details><summary>Explanation of command</summary>
<p><code>sed</code> is a non-interactive editor for character streams, built around regular expressions.
The <code>-i</code> flag instructs <code>sed</code> to edit files in-place, rather than writing the edited character stream to the standard output.
The expression <code>'s/^# //'</code> tells <code>sed</code> to substitute occurrences of "<code># </code>" at the beginning of lines with nothing (in other words, delete them).
By adding <code>/- non-free/</code> to the beginning of that expression, it tells it to only apply the substitution on lines containing matches for the pattern <code>- non-free</code>.
This results in only that one line being uncommented, or no change if it has already been uncommented.</p>
</details>

3. Enable the VSCode repository by running the following:

```sh
extrepo enable vscode
```

4. Update apt metadata cache to load the info from the new repository, then install VScode, by running the following:

```sh
apt update
apt install code
```

Note that in this example, it might be better to install VSCodium, which is built from most of the same code as VSCode, but is fully compliant with the Debian Free Software Guidelines, and does not include telemetry-gathering - in other words, it's fully open-source and privacy-friendly. The process for that is similar - just replace all instances of `code` with `codium` in the above process, and optionally skip step 2, as it's unnecessary. I personally would recommend VSCodium in any case that you do not require a specific proprietary VSCode extension, as many proprietary extensions' licenses restrict them to use only with the official VSCode binaries.

#### Manually Adding a Repository

If at all possible, I would recommend using [`extrepo`](#extrepo), as explained above. If extrepo does not include the repository, **and you trust the repository's administrator/s**, you can add it manually. Typically, the administrator/s of the repository will have instructions available online, but, in my experience, they will typically recommend the use of tools like `apt-key`, or manually adding the key to `/etc/apt/trusted.gpg.d`, which has the same security implications and should be considered bad practice. In such cases would recommend referring to the instructions, but not following them directly. The good news is that this seems to be changing - of the 4 repositories I looked at while writing this section, 2 of them instructed users to save the keys to `/usr/share/keyrings` instead. That is an improvement, but it's still not good practice to do that either, as paths in `/usr` (other than `/usr/local`) should be used for system packages exclusively.

There are 3 steps to adding a repository manually: 

1. Acquire the GPG key for the repository.
2. Create the repository configuration file
3. Update the apt cache

##### Acquiring GPG key

Create a directory to store the key/s in if it doesn't yet exist, with permission mode 755:

```sh
install -dm755 /etc/apt/keyrings
```

*Note:* a previous version of this page used `/usr/local/share/keyrings`, which I though made sense as `/usr/local/share` is the equivalent of `/usr/share` that is expected to be managed by the local admin, as I alluded to above. It turns out the man page for Debian Bookworm, which is the testing release at time of writing, specifically recommends using `/etc/apt/keyrings` for that purpose, so I've updated this page to reflect that.

Determine the key to add. For example, the [PowerShell installation instructions for Debian 11](https://learn.microsoft.com/en-us/powershell/scripting/install/install-debian?view=powershell-7.3#installation-on-debian-11-via-package-repository) contains the following command:

```sh
# Import the public repository GPG keys
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```

I do not recommend running that command - instead, what I'd run is the following:

```sh
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor --output /etc/apt/keyrings/microsoft-apt-repo.gpg
```

Change the URL and `--output` path as appropriate. If the URL ends with "`.gpg`", you don't need to `--dearmor` it, and should instead just delete `| gpg --dearmor `, to pass the `--output` argument straight to `curl`.

##### Creating the Repository Configuration File

Create a file in **/etc/apt/sources.list.d** with the extension **.list** or **.sources**. The format for these files is described in the **sources.list(5)** man page - run `man 5 sources.list` to read it locally, or view it online at [manpages.org](https://manpages.org/sourceslist/5).

The short version is that you'll probably want to fill in one of the following templates, replacing `${shell_style_variables}` with the appropriate values:

###### Template for .list format

```debsources
deb [arch=${dpkg_architectures} signed-by=${path_to_signing_key}] ${repo_uri} ${suite} ${components}
```

###### Template for .sources format

```
Types: deb
Architectures: ${dpkg_architectures}
Signed-By: ${path_to_signing_key}
URIs: ${repo_uri}
Suites: ${suite}
Components: ${components}
```

The appropriate values can be determined as followed:
* `${dpkg_architectures}`: the architectures supported in the repository. If not specified in the original instructions, omit the entire `arch=${dpkg_architectures} ` part of the **.list** format template. For the **.list** format, this list is comma-separated, while for the **.sources** format, this list is space-separated.
* `${path_to_signing_key}`: this is the path you saved the key to in the previous step
* `${repo_uri}`: The URI of the repository. Copy from the original instructions.
* `${suite}`: most packages are built to work with a particular version of Debian, and most repositories have different versions for different releases. In general, this will be the codename of the particular Debian release a repository is built for, though there are exceptions to this. Copy from the original instructions.
* `${components}`: a space-separated list of components to use. Copy from the original instructions.

##### Example: Installing PowerShell on Debian Bullseye

Adapted from [Powershell: Installation § Debian 11 (Bullseye)](../../Powershell/Installation.md#debian-11-bullseye)

```sh
# ensure that curl and gpg commands are installed
if ! which curl gpg >/dev/null; then apt update && apt install -y curl gnupg; fi

# ensure that the keyring directory exists
test -d /etc/apt/keyrings || install -dm755 /etc/apt/keyrings

# download the GPG signing key
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor --output /etc/apt/keyrings/microsoft-apt-repo.gpg

# use the .list format
cat >/etc/apt/sources.list.d/microsoft-apt-repo.list <<EOF
deb [arch=amd64 signed-by=/etc/apt/keyrings/microsoft-apt-repo.gpg] https://packages.microsoft.com/repos/microsoft-debian-bullseye-prod bullseye main
EOF

# update apt cache to add the new repo contents
apt update
```
