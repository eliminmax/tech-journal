<!--
SPDX-FileCopyrightText: 2023 - 2024 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# GlusterFS

<!-- vim-markdown-toc GitLab -->

* [Setup](#setup)
  * [Installation](#installation)
  * [Configuring a GlusterFS Volume](#configuring-a-glusterfs-volume)
* [Mounting a GlusterFS Volume](#mounting-a-glusterfs-volume)

<!-- vim-markdown-toc -->

GlusterFS is a distributed filesystem, which allows you to split a volume across multiple hosts, or replicate data to multiple hosts.

## Setup

For the sake of this example, I'm going to explain how to re-create my setup from my [Redundancy/High Availability project](https://github.com/eliminmax/cncs-journal/wiki/Working-Notes%3A-SEC440%3A-Redundant-Infrastructure). That means I'm working with 3 nodes, all running Ubuntu 22.04, They are as follows:

| hostname | ip address  |
|----------|-------------|
| `u1-eli` | `10.0.6.21` |
| `u2-eli` | `10.0.6.22` |
| `u3-eli` | `10.0.6.23` |

### Installation

You'll need to install the GlusterFS server software on all of the nodes, and enable and start the `glusterd` service.

For example, on Debian or Ubuntu:


```sh
apt install glusterfs-server
systemctl enable --now glusterd
```

Be sure to allow TCP connections from other nodes to port 24009, and from both other nodes and clients to port 60116.

### Configuring a GlusterFS Volume

On each node, create an empty directory to use as a "brick" for the GlusterFS volume - I'd recommend doing that on a dedicated partition, but that's beyond the scope of this wiki page. In my case, I set it up at `/data/glusterfs/nextcloud-store/brick`.

Once you've done that, run the following commands on u2:

```sh
gluster peer probe u2-eli
gluster peer probe u3-eli
```

Note that the hostnames are in the host files for all three nodes, so the address could be resolved

Now, the connection has been established, and we can create the actual volume.

```bash
gluster volume create gv0 replica 3 u{1..3}-eli:/data/glusterfs/nextcloud-store/brick
```

The above command creates a volume called "gv0", and specifies that data should be replicated to 3 bricks for redundancy, then defines 3 bricks to use - the `u{1..3}-eli:/data/glusterfs/nextcloud-store/brick` is expanded by `bash` into `u1-eli:/data/glusterfs/nextcloud-store-brick u2-eli:/data/glusterfs/nextcloud-store-brick u3-eli:/data/glusterfs/nextcloud-store-brick`. Specifying each one individually is more portable, but obviously slower.

Finally, start the volume with `gluster volume start gv0`. Now, the volume is ready to be mounted.

## Mounting a GlusterFS Volume

You need to install a filesystem driver for GlusterFS to mount a volume. On Linux, one filesystem driver is built on FUSE (Filesystem in USErspace), specifically used for GlusterFS. Alternatively, you can mount it as an NFS volume or CIFS (used for file shares on Windows). For my Redundancy/High Availability project, I mounted the volume on 2 different web servers, both running CentOS 7, using the dedicated gluster client.To do that, I needed to install `centos-release-gluster41` package and enable the extra repos it provided, then install the `glusterfs-client` package.

To mount the volume, I first edited `/etc/fstab`, which defines automatic mounts. I then ran `systemctl daemon-reload`, because SystemD automatically creates mount units based on the fstab file at startup, but needs to be told to update it with that command if the fstab file is edited later.

The line I added was as follows:

```fstab
u1-eli:gv0 /var/www/html/nextcloud glusterfs defaults,_netdev 0 0
```

This indicates that it should mount `u1-eli:gv0` on `/var/www/html/nextcloud`, and tells it that it is a GlusterFS volume, so that it uses the appropriate driver. The next field indicates to use the default mount options, and that it's a network drive, so that the userland tool responsible for mounting it can wait for a network connection. The first of the 2 zeroes indicate not to include it when backing disks up with `dump`, which is an ancient tool. The second indicates not to check filesystem integrity when booting up.
