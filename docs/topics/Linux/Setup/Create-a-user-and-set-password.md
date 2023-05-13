
> <details>
> <summary>Meaning of different command prompts</summary>
> Unix/Linux: <code>$</code>: can be run as normal user<br>
> Unix/Linux: <code>#</code>: must be run as root (or with <code>sudo</code>)<br>
> Windows: <code>></code>: Command Prompt or PowerShell<br>
> Windows: <code>PS></code>: PowerShell only<br>
> Unix/Linux and Windows: <code>$/></code>,<code>#/></code>: Works in Windows and Unix/Linux.
> </details>

## Create a User

run the following, replcing `<PLACEHOLDER_VALUES>` as needed:

 `# useradd -d <HOMEDIR> -G <GROUPS> -s <SHELL> -m <USERNAME>`

   * `<HOMEDIR>`: path to the new user's home directory
   * `<GROUPS>`: should be a comma separated list of groups
   * `<SHELL>`: shell that the user defaults to on login. If you're not sure, go with `/bin/bash`
   * `<USERNAME>` name of the new user

## Set Password

To set the password for the new user, use the `passwd` command.

`# passwd <USERNAME>`

  * `<USERNAME>`: name of the user who's password is changing

If you are setting/resetting someone else's password, pass the `-e` flag to make change the password when they log in

`# passwd -e <USERNAME>`
