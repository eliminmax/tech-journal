<!--
SPDX-FileCopyrightText: 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Powershell: Configuration

<!-- vim-markdown-toc GitLab -->

* [$PROFILE](#profile)

<!-- vim-markdown-toc -->

## $PROFILE

```powershell
# Create the file if it does not already exist
if (-Not (Test-Path -Path $PROFILE -PathType Leaf)){
    New-Item -ItemType File -Path $PROFILE -Force
}
# Note: The -Force flag will create the directories if needed, but will overwrite existing files
```

Keep in mind that Powershell may use different `$PROFILE` files for different versions or contexts - for example, on Windows, for Powershell Core, it's located at `$HOME\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`, or either `$HOME\Documents\WindowsPowerShell\Profile.ps1` or `$HOME\My Documents\WindowsPowerShell\Profile.ps1` for Windows Powershell. The version of Powershell that runs within VSCode and VSCodium would be `$HOME\Documents\PowerShell\Microsoft.VSCode_profile.ps1`.

To edit the profile, you can open it with your text editor of choice. In my case, I'd start a Powershell session with the `pwsh` command, and run `vim $PROFILE`.
