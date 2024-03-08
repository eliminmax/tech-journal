# VRRP

<!-- vim-markdown-toc GitLab -->

* [VyOS](#vyos)
* [keepalived](#keepalived)

<!-- vim-markdown-toc -->

VRRP, the Virtual Router Redundancy Protocol, is a protocol designed to coordinate between multiple routers, to enable failover should one go down. A VRRP setup essentially creates a fake router with its own IP address, and different actual routers can act as the fake router. Each actual router involved must have a different priority number configured, and the highest-priority router is the one that acts as the virtual router. If it goes down, the next-highest steps in, allowing for smooth fail-over.

While designed for use by routers, there is no reason it can't be used for other network services - in fact, it often is. There are 2 implementations covered in this document - VyOS VRRP, and keepalived

## VyOS

Setup on VyOS is simple. All you need to do is [enter configuration mode](../VyOS.md#entering-configuration-mode), run the configuration commands listed below, then [commit and save the changes](../VyOS.md#save-configuration-changes).

For the sake of example, I'm assuming `router0` and `router1` are both fully configured, with the following interfaces and addresses:


| router  | eth0 DMZ, /24 | eth1 LAN, /20 | eth2 MGMT, /27 |
|---------|---------------|---------------|----------------|
| router0 | 10.20.2.201   | 10.0.5.2      | 10.200.0.2     |
| router1 | 10.20.2.202   | 10.0.5.3      | 10.200.0.3     |

I will set up the following 3 VRRP addresses
| vrid 10, DMZ, /24 | vrid 20, LAN, /20 | vrd 30, MGMT, /27 |
|-------------------|-------------------|-------------------|
| 10.20.2.200       | 10.0.5.1          | 10.200.0.1        |

On router0, run the following in `configure` mode:

```
set high-availability vrrp group DMZ-VRRP address 10.20.20.200/24
set high-availability vrrp group DMZ-VRRP interface eth0
set high-availability vrrp group DMZ-VRRP priority 200
set high-availability vrrp group DMZ-VRRP vrid 10
set high-availability vrrp group LAN-VRRP address 10.0.5.1/20
set high-availability vrrp group LAN-VRRP interface eth1
set high-availability vrrp group LAN-VRRP priority 200
set high-availability vrrp group LAN-VRRP vrid 20
set high-availability vrrp group MGMT-VRRP address 10.200.0.1/27
set high-availability vrrp group MGMT-VRRP interface eth2
set high-availability vrrp group MGMT-VRRP priority 200
set high-availability vrrp group MGMT-VRRP vrid 30
```

On router1, the commands are nearly identical, except set the `priority` to 100 instead.

## keepalived

This is an implementation of VRRP designed for non-router systems, often used alongside [HAProxy](./HAProxy.md).

In my [Redundancy/High Availability project](https://github.com/eliminmax/cncs-journal/wiki/Working-Notes%3A-SEC440%3A-High-Availability-Project), I adapted [this tutorial](https://kifarunix.com/configure-highly-available-haproxy-with-keepalived-on-ubuntu-20-04/)'s configuration of HAProxy+keepalived.

The most notable part of the configuration file was that it included a command used to ensure that HAProxy was still running.

The template for the configuration file is as follows:

```conf
# vi:ft=conf:et:sw=4:ts=4:sts=4
{% raw %}
global_defs {
    enable_script_security
    script_user keepalived
}

vrrp_script chk_haproxy {
    script "/usr/bin/killall -0 haproxy"
    interval 2
    weight 2
}

vrrp_instance LB_VIP {
    interface ens160
    state {{ keepalived.state }}
    priority {{ keepalived.host.priority }}
    virtual_router_id 51

    authentication {
        auth_type AH
        auth_pass {{ keepalived.password }}
    }
    unicast_src_ip {{ keepalived.host.ip }}
    unicast_peer {
        {{ peer.ip }}
   }

    virtual_ipaddress {
        {{ keepalived.ip }}
    }


    track_script {
        chk_haproxy
    }
}
{% endraw %}
# adapted from https://kifarunix.com/configure-highly-available-haproxy-with-keepalived-on-ubuntu-20-04/#ipforwardingandnon-localbind
```
