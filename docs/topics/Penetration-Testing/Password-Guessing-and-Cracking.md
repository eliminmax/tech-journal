# Password Guessing and Cracking - Tools and techniques

<!-- vim-markdown-toc GFM -->

* [Password Guessing](#password-guessing)
  * [CeWL](#cewl)
  * [RSMangler](#rsmangler)
  * [Hydra](#hydra)
* [Password Cracking](#password-cracking)
  * [/etc/shadow](#etcshadow)
    * [Getting password hashes from /etc/shadow](#getting-password-hashes-from-etcshadow)
      * [Explanation](#explanation)
* [Password Cracking](#password-cracking-1)
  * [unshadow](#unshadow)
  * [John the Ripper](#john-the-ripper)
  * [Hashcat](#hashcat)

<!-- vim-markdown-toc -->

## Password Guessing

### CeWL

The Custom Word List generator can be used to generate wordlists based on the contents of target websites. It parses a webpage and any pages it links to, up to a given depth.

Example usage from [Kali docs](https://www.kali.org/tools/cewl/)
```bash
cewl -d 2 -m 5 -w docswords.txt https://example.com
```

Breakdown of example:
* `-d 2`: spider to a maximum depth of 2
* `-m 5`: minimum word length of 5
* `-w docswords.txt`: write wordlist generated to docswords.txt
* `https://example.com`: start spidering from this url

### RSMangler

Mangle a given wordlist to create a much longer one

Specify input file with `-f`, output file with `-o`, minimum password size with `-x`, and maximum password size with `-m`. Other options disable specific mangling techniques - list them with `--help` or `-h`.

### Hydra

Given a wordlist and a target, this tool will attempt all passwords in the wordlist.

```sh
# try to access admin@192.0.2.5 over ssh using passwords in wordlist, with up to 4 parallel tasks
hydra -l admin -P wordlist -s 22 -f 192.0.2.5 ssh -t 4
# try to access http://192.0.2.5:8080/admin/ protected by http basic authentication using the username user and passwords in wordlist
hydra -l user -P wordlist -s 8080 -f 192.0.2.5 http-get /admin/
# a more compact form of the previous command:
hydra -l user -P wordlist http-get://192.0.2.5:8080/admin/
```

## Password Cracking

### /etc/shadow

The /etc/shadow file contains password data. It's extensively documented in the [SHADOW(5)](https://manpages.org/shadow/5) man page (accessible with the command `man 5 shadow`). Each line the following nine fields, in order, separated by colons (`:`):

1. **login name** - from the man page: "It must be a valid account name, which exist on the system."
2. **encrypted password** - the way it's stored is detailed in the [CRYPT(3)](https://manpages.org/crypt/3) man page; run `man 3 crypt` to read it. If it's empty, then no password is needed to log in. If it's an invalid CRYPT(3) hash, it's impossible to log in. System accounts will typically have `*` or `!` in this field for this reason.
3. **date of last password change** - "date" here meaning the number of days since Jan 1, 1970. If it's 0, that means that it must be changed on the next login. If it's empty, then that means that the functionality related to password age is disabled.
4. **minimum password age** - the user must wait at least this many days between password changes. If empty, interpret it as 0.
5. **maximum password age** - if the password is older than this, the user should be prompted to change it when they next log in. If lower than minimum age, the user cannot change their password
6. **password warning period** - warn user to change password this many days before it expires. If empty or 0, no warning is issued
7. **password inactivity period** - disable account this many days after it expires if the password has not been changed. Do not disable if it's empty or 0.
8. **account expiration date** - How many days after Jan 1, 1970 this account expires. If empty, it does not expire. If 0, it can be interpreted as having expired on Jan 1, 1970 or as not expiring. Do not set to 0.
9. **reserved field** - quoting the man page, "This field is reserved for future use."

#### Getting password hashes from /etc/shadow

On a Linux system, as root, run `awk '/^[^:]+:\$/ {print $0}' /etc/shadow` to print only lines containing password hashes, because CRYPT(3) hashes start with a dollar sign.

##### Explanation

awk is a text processing domain-specific language that is a standard part of *nix systems. What this one-liner does is print all lines containing a regular expression match.

`/^[^:]+:\$/` means to execute the following command on any line that begins with one or more non-colon characters, followed by a colon, and a dollar sign. (The first `^` means the start of the line, the `[^:]+` means one or more non-colon characters, the `:` is just a literal colon, with no special meaning, and the dollar sign would normally represent the end of the line, so to avoid that, it's backslash-escaped, resulting in `\$`. The dollar sign is the

`{print $0}` simply prints the entire line.

I tested this with 3 different implementations of `awk` (GNU's `gawk`, `mawk`, and `busybox awk`)

## Password Cracking

### unshadow

`unshadow /etc/passwd /etc/shadow >unshadowed` combines the contents of the `/etc/passwd` and `/etc/shadow` files into a file called `unshadowed`, for use with John the Ripper or Hashcat

### John the Ripper

John the Ripper cracks passwords. That's his one and only job, passion, and talent. Summon him with the command `john unshadowed`. To give him a wordlist, rather than letting him bring his own, use the command `john -wordlist:/path/to/list unshadowed`.

### Hashcat

Like john, but ***much*** faster (in my experience). Took only about 40-50 seconds to do what took John the Ripper about 20-25 minutes. That said, it missed one of the three passwords, but `john` got all three.

`hashcat -m 1800 -a 0 -o cracked_passwords unshadowed wordlist` will run a basic dictionary attack on `unshadowed`, targeting SHA-512 Unix passwords, using `wordlist`, and output the results to the `cracked_passwords` file. The `-m 1800` tells it to use hash mode 1800. The man page lists what hash types are supported, and what their numeric modes are. 1800 is SHA-512(Unix). An online version of that is available [here](https://manpages.org/hashcat).
