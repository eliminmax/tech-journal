# TLS/SSL Certificate Signing

> <details>
> <summary>Meaning of different command prompts</summary>
> Unix/Linux: <code>$</code>: can be run as normal user<br>
> Unix/Linux: <code>#</code>: must be run as root (or with <code>sudo</code>)<br>
> Windows: <code>></code>: Command Prompt or PowerShell<br>
> Windows: <code>PS></code>: PowerShell only<br>
> Unix/Linux and Windows: <code>$/></code>,<code>#/></code>: Works in Windows and Unix/Linux.
> </details>

## OpenSSL

From the directory you want to save your certificates to, such as `/etc/ssl`, do the following:

### Generate a new key pair and certificate for for.example:
`$ openssl req -new -newkey rsa:2048 -nodes -keyout for.example.key -out for.example.csr`

***CAREFULLY*** enter the information that it prompts you for.

Copy **the CSR file** (and not the private key file) over to the Certificate Authority, and
copy the signed certificate to `for.example.crt`.
