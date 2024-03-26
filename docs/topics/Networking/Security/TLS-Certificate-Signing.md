<!--
SPDX-FileCopyrightText: 2022 - 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# TLS/SSL Certificate Signing

## OpenSSL

From the directory you want to save your certificates to, such as `/etc/ssl`, do the following:

### Generate a new key pair and certificate for for.example:
`openssl req -new -newkey rsa:2048 -nodes -keyout for.example.key -out for.example.csr`

***CAREFULLY*** enter the information that it prompts you for.

Copy **the CSR file** (and not the private key file) over to the Certificate Authority, and
copy the signed certificate to `for.example.crt`.
