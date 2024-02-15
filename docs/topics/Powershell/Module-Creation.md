# Powershell - Module Creation

<!-- vim-markdown-toc GitLab -->

* [Set up module](#set-up-module)
  * [Creating the module file](#creating-the-module-file)
  * [Creating the Module Manifest](#creating-the-module-manifest)
  * [Module Path](#module-path)

<!-- vim-markdown-toc -->

## Set up module

For the sake of example, I'm going to create a simple module that has one function, called "Test-Foo", which takes a single parameter called FooString, and prints a message indicating whether or not FooString is set to "foo". Not particularly useful, but it illustrates the module creation process.

### Creating the module file

Create a file called `foo.psm1`, and add the following:

```powershell
function Test-Foo {
    <#
    .SYNOPSIS
    Tests whether FooString is "foo", and prints a message

    .DESCRIPTION
    Tests whether FooString is "foo", and prints a message indicating the result. Not particularly useful.

    .PARAMETER FooString
    The string to test
    .EXAMPLE
    PS> Test-Foo -FooString "foo"
    FooString is foo.

    .EXAMPLE
    PS> Test-Foo -FooString "bar"
    FooString isn't foo.

    #>
    param (
        [Parameter(Mandatory=$True)] [String] $FooString
    )
    if ( $FooString -eq "foo" ) {
        Write-Host "FooString is foo."
    } else {
        Write-Host "FooString isn't foo."
    }
  }
```

The docstring is used by the `Get-Help` cmdlet to provide help for the function. More details can be found by running `Get-Help about_Comment_Based_Help`. More about advanced parameter usage can be found with `Get-Help about_Functions_Advanced_Parameters`.

### Creating the Module Manifest

There's a cmdlet to create the module manifest, appropriately called `New-ModuleManifest`. It has many optional parameters, but for this example, I'd run the following from the same directory as `foo.psm`:

```powershell
New-ModuleManifest -RootModule 'foo.psm1' -Path 'foo.psd1' -Author 'Eli Array Minkoff' -CompanyName 'eliminmax' -Copyright 'Eli Array Minkoff'
```

This creates the manifest file. To import it, run `Import-Module ./foo.psd1`. If you've already imported it, you can re-import it using the `-Force` flag.

### Module Path

Powershell will search specific directories for modules to import, so you don't need to specify the path to the file. The directories are specified with the `$Env:PSModulePath` environment variable. On Windows, they are separated by semicolons (`;`), but on Linux and macOS, they are separated by colons (`:`). It searches them in order.

You can add directories in your Powershell profile, which is run at the start of every Powershell session, though I'd recommend checking that the directory is not already present first. See [Powershell: Configuration § $PROFILE](./Configuration.md#profile) for instructions on setting that up.

Once set up, you can add the following to your profile, to append a directory to the `$Env:PSModulePath` variable if it is not already included. Note that I am using the Linux/macOS path separator in this.

```powershell
function New-PSModulePathLocation ([string] $ModulePath) {
    if ( $ModulePath -in $Env.PSModulePath.Split(":")) {
        Write-Warning "$ModulePath already included in Env:PSModulePath."
    } else {
        $Env:PSModulePath += ":$ModulePath"
    }
}
New-PSModulePathLocation "/path/to/module/directory"
```
