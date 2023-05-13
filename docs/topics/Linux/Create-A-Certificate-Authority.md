# Linux: Create a Certificate Authority

> <details>
> <summary>Meaning of different command prompts</summary>
> Unix/Linux: <code>$</code>: can be run as normal user<br>
> Unix/Linux: <code>#</code>: must be run as root (or with <code>sudo</code>)<br>
> Windows: <code>></code>: Command Prompt or PowerShell<br>
> Windows: <code>PS></code>: PowerShell only<br>
> Unix/Linux and Windows: <code>$/></code>,<code>#/></code>: Works in Windows and Unix/Linux.
> </details>

## Easy-RSA

### Rocky Linux 8

#### Installation/Setup
This was adaped from [A DigitalOcean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-ca-on-centos-8)

0. Install Required Software

   * If needed, enable Extended Packages for Enterprise Linux: `# dnf install epel-release`
   * Install Easy-RSA: `# dnf install easy-rsa`

1. Set Up CA - ***DO NOT RUN AS ROOT***

   * create the directory: `$ mkdir ~/easy-rsa`
   * create symbolic links to the files in /usr/share/easy-rsa/3:
`$ ln -s /usr/share/easy-rsa/3/* ~/easy-rsa`
   * block group/other users from accessing ~/easy-rsa: `$ chmod 700 ~/easy-rsa`

**NOTE:** Some tutorials will tell you to copy the Easy-RSA files from /usr/share/easy-rsa/3
to ~/easy-rsa, but symlinking them allows updates to `easy-rsa` to be reflected immediately.

2. Set up variables

   * Create a file at ~/easy-rsa/vars, and set the following default values:
```
set_var EASYRSA_REQ_COUNTRY   ""
set_var EASYRSA_REQ_PROVINCE  ""
set_var EASYRSA_REQ_CITY      ""
set_var EASYRSA_REQ_ORG       ""
set_var EASYRSA_REQ_EMAIL     ""
set_var EASYRSA_REQ_OU        ""
set_var EASYRSA_ALGO          "ec"
set_var EASYRSA_DIGEST        "sha512"
```

For the `EASYRSA_REQ_*` values, set them as follows:

variable               | value
-----------------------|---------------------------------------------------------
`EASYRSA_REQ_COUNTRY`  | 2-letter Country Code
`EASYRSA_REQ_PROVINCE` | Full name of state or province (e.g. California, not CA)
`EASYRSA_REQ_CITY`     | Name of the city
`EASYRSA_REQ_ORG`      | Name of the organization
`EASYRSA_REQ_EMAIL`    | Email address
`EASYRSA_REQ_OU`       | Organizational Unit

3. Build the Certificate Authority

   * from the easy-rsa directory, run `$ ./easyrsa build-ca`
      * this will require a password to use - if you want to set it up without a password, append `nopass` to the command

4. Distribute the Public Certificate
   * The public certificate will be stored in the file ~/easy-rsa/pki/ca.crt.

#### Signing Certificates

0. Copy the certificate signing request to the /tmp directory.

1. From the easy-rsa directory, run `$./easyrsa import-req /tmp/for.example.csr for.example`
   * The last line will contain the path to the certificate

2. Copy the signed certificate back to the web server.

## Add your Certificate Authority to the system trust

### Debian and derivatives:

0. Copy the root authority to /usr/local/share/ca-certificates/

1. run `# update-ca-certificates`

### RHEL and derivatives:

*Similar to the previous section, but with a different path and command*

0. Copy the root authority to /etc/pki/ca-trust/source/anchors/

1. run `# update-ca-trust`

### Windows

0. Download the root certificate, right click it, and click **Install Certificate**

1. Save it to **Trusted Root Certificate Authorities**

### Note about Firefox

By default, Firefox ignores the system's Certificate Authorities, and uses its own trust. You
can find instructions for adding certificates [in the Firefox for Enterprise documentation](https://support.mozilla.org/en-US/kb/setting-certificate-authorities-firefox).
