<!--
SPDX-FileCopyrightText: 2022 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Wireguard VPN Setup and Configuration

Wireguard is a VPN protocol that was originally made for Linux, and it has support built into the Linux kernel.

## Linux Configuration

### Dependencies

For the sake of this example, I am assuming that the `ip` command is available.
On Debian and Ubuntu, it is part of the `iproute2` package.

There are 2 components to Wireguard that must be installed - the kernel-level support and the command-line tools. The former may be compiled in to the Linux kernel already, depending on the build configuration. The latter will probably need to be manually installed.

On Debian and Ubuntu, the `wireguard` metapackage takes care of installing these dependencies, so simply install that package.

### Creating a Wireguard interface

```sh
ip link add wg0 type wireguard
```

### Setting an IP address

This example uses the arbitrarily-chosen IP address `192.168.20.21` in a `/24` network.

```sh
ip address add dev wg0 192.168.20.21/24
```

### Generating a key pair

```sh
wg genkey > wg0-priv 
wg pubkey < wg0-priv > wg0-pub
```

### Adding peers

Given the **public** key, the IP address and port of a peer's Wireguard setup, and a public IP or a hostname of that peer, add the peer with the following command.
```sh
wg set wg0 peer anVzdCBiYXNlNjQtZW5jb2RlZCB0aGlzIHN0cmluZy4= allowed-ips 192.168.20.22/32 endpoint peer.example:51900
```

### All Together Now

*As soon as I came up with this section name, I started to hear the chorus from the Beatles song playing on loop in my head with no end in sight.*

For the sake of this example, I am going to connect 2 Ubuntu 22.04 systems, and will assume that the `wireguard` package is installed already. I'll call the systems `yellow` and `submarine`. *No idea where those names came from.* Let's say `yellow`'s public IP is 192.0.2.126, and `submarine` is behind some kind of NAT setup and has no public IP. The Wireguard addresses will be `10.20.30.40/24` and `10.20.30.41/24` for `yellow` and `submarine` respectively.

The commands in this system should be run as root. I'd recommend using `sudo -i`, `su -`, or something similar to start a shell as root.

The first part here is the same on both systems.

```sh
ip link add wg0 type wireguard
cd /etc/wireguard
wg genkey > wg0-priv
wg pubkey < wg0-priv > wg0-pub
```

Next, set the IP addresses on each host's `wg0` interface

On `yellow`:

```sh
ip address add dev wg0 10.20.30.40/24
```

On `submarine`:

```sh
ip address add dev wg0 10.20.30.41/24
```

Now, configure `wg0` to use the `wg0-priv` key created previously, and configure both hosts to listen on port 51900 (UDP), and bring up the `wg0` interface This is the same on both systems. Note that you'll need to ensure that there are no firewall rules blocking this port.

```sh
wg set wg0 privkey /etc/wireguard/wg0-priv
wg set wg0 listen-port 51900
ip link set wg0 up
```

## Other systems

The following contains an example setup that connects a Windows 10 device to a VyOS device. While VyOS is technically a Linux system, it has its own approach to configuration.

### Example: Wireguard connection from Windows to VyOS

For this example, we'll allow connections from a Windows 10 device called `remote` to a VyOS device `fw`, to access the `10.10.20.0/24` network (I am *killing it* in terms of creative names tonight)

<details>
<summary><em><strong>Disclaimer about the use of keys here</strong></em></summary>
<em>You should NEVER, and I mean <strong>NEVER,</strong> share any sort of private key - the key pairs I am using here was generated exclusively to be used on this page, and is not, and will never be, used in any sort of real-world environment, even a virtualized one. I spun up a generic Debian Docker container, installed <code>wireguard-tools</code>, generated the two example key pairs, then deleted the container.</em>
</details>

#### On `remote`:

0. Download and install Wireguard from [the official link](https://www.wireguard.com/install/), and create a new tunnel.
1. Save the public key (for this example, let's say it's `cYPncKxxCovLYXrvdFoUJL/y2AK7FUOOfYkQE8O+bm0=`)

#### On `fw`:

0. **Config mode:** run the following

```
set interfaces wireguard wg0 address '10.10.10.1/24'
set interfaces wireguard wg0 peer ctrl allowed-ips '10.10.10.10/32'
set interfaces wireguard wg0 peer ctrl public-key 'cYPncKxxCovLYXrvdFoUJL/y2AK7FUOOfYkQE8O+bm0='
set interfaces wireguard wg0 port '54321'
run generate pki wireguard key-pair install interface wg0
```

1. Save the public key (for this example, let's say it's `bHit6VZXq8D+xwgaFMQiWGWqNDXPLnZRdqfwV1H44Ro=`)

#### Back to `remote`:

Edit the tunnel, so that it reads similarly to the one below

```ini
[Interface]
PrivateKey = kNuGO3KZeqXKj0SgQEdGNVDVqCjeIwE1hazaBVcni1k=
Address = 10.10.10.10/32

[Peer]
PublicKey = bHit6VZXq8D+xwgaFMQiWGWqNDXPLnZRdqfwV1H44Ro=
AllowedIPs = 10.10.10.1/32, 10.10.20.0/24
Endpoint = 10.10.10.1:54321
```

2. Click the button to activate the tunnel - it should be ready to go now
