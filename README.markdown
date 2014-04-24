
Easy Firewall Script
====================

Ever wanted a nifty firewall but have no time to learn the Linux Netfilter interface? This script will do the work.

Features
--------

* IPv4 support
* Optional IPv6 support
* Ping throttling, or Ping disable
* Optional SSHGuard support
* Avoiding many types of port scans
* Logging for Port Scan detection
* Optional Multiple Port Knocking
* Optional Multi-home support
* Optional trusted interface specification

Invocation
----------

Configuration is a single command line invocation. With no arguments, the firewall will block all incoming connections.

To display a help message:

	iptabs --help

To allow SSH over IPv4 and IPv6:

	iptabs --enable-tcp ssh

By default pings are throttled to protect from flooding. To disable pings completely:

	iptabs --drop-ping

By default ports allowed with IPv4 are also allowed with IPv6. To disable IPv6:

	iptabs --drop-ipv6

But really, IPv6 is coming, try to be ready for it.

Services can be specified by name or number. Service names must match the information in `/etc/services`. Most Linux distributions come with a fairly complete list.

To enable a list of services, i.e. SSH on port 22, HTTP on port 80 and HTTPS on port 443

	iptabs --enable-tcp ssh,www,https

UDP services can also be enabled:

	iptabs --enable-udp echo

Any combination of TCP and UDP ports can be specified:

	iptabs --enable-tcp ssh,ftp,ftp-data --enable-udp tftp

Multiple interfaces are supported via the `--interface` option. The interface will take effect for rules specifed after it. As a common example, assume `eth0` is the private network, and `eth1` is the public network. SSH should be allowed only on the private interface, but http and https should be allowed on all interfaces.

	iptabes --interface eth0 --enable-tcp ssh --interface all www,https

Or, since `--interface all` is the default:

	iptabes --enable-tcp http,https --interface eth0 --enable-tcp ssh

No validation is done to the interface specifier, so watch your output carefully, iptables and ip6tables will produce an error if the interface specifier is not correct.

If SSHGuard is installed, enable support for it:

	iptabs --sshguard

If fwknop server support is desired on the server, enable it:

	iptabs --fwknop

Note that using fwknop as a client requires no special support.

By default SYN protection is enabled, XMAS packets are dropped, NULL packets are dropped, and FIN scans are logged for programs like PSAD. Any of these can be optionally disabled.

To dump the netfilter commands to a file:

	iptabs --dump-to netfilter.sh

This will write the commands to the file `netfilter.sh` instead of executing them. This is particularly handy on servers that do not have Perl installed.

Port Knocking
-------------

Four part port knocking can also be enabled. Since it is Netfilter only, no cryptographic or stealth support is available. Try something like fwknop for that.

To enable port knocking to open SSH via the sequence 444, 333, 222, 111:

	iptabs --knock ssh:444:333:222:111

To connect via SSH from another computer, first connect something to port 444, then port 333, then 222, then 111, then open the SSH connection. Several programs are available for this purpose, Google is your friend.

Each service can have its own port knock sequence. Remember not to enable SSH via `--enable-tcp` if port knocking is set up for it.

If some sort of external port scan detector is in use port knocking can trigger it. If this occurs, disable port knock logging:

	iptabs --knock ssh:444:333:222:111 --no-knock-log

Specified by Network/Host
-------------------------

A list of allowed ports can also be specified by a network or host string. For example, to allow SSH only from the 192.168.1.0 network:

	iptabs --by-spec 192.168.1.0/24:tcp:ssh

Multiple ports may be specified:

	iptabs --by-spec 192.168.1.0/24:tcp:ssh,http,443

Multiple invocations of `--by-spec` are allowed. The traffic is subject to the other rules, such as SSHGuard and port scan detection.

Trusted Interface
-----------------

Trusted interfaces can be marked via `--trusted-iface` with a single interface, or a comma separated list of interfaces. The loopback interface is automatically marked as trusted, so do not bother adding it with this option.

Example of a single trusted interface:

	iptabs --trusted-iface eth1

Example of multiple trusted interfaces:

	iptabs --trusted-iface eth1,eth2,eth4

NAT/IPSec
---------

IPSec and NAT are supported, in any combination.

To enable NAT from private network on eth0 to public eth1

    iptabs --nat eth0:eth1

To enable IPSec tunneling

    iptabs --ipsec

To enable IPSec tunneling with NAT on the private network (Note the reversal of public eth1 and private eth0)

    iptabs --ipsec --nat eth1:eth0

For strongswan with NAT

    iptabs --ipsec --nat eth1:eth0 --enable-udp 500,4500

Untrusted Networks/Addresses
----------------------------

Problematic networks or hosts can be blacklisted. Multiple hosts or networks may be specified with each invocation. IPv4 and IPv6 are both supported.

For example, some spammer is throwing traffic at the mail server from 192.168.1.1

    iptabs --blacklist 192.168.1.1/32

Or the spammer controls a whole network

    iptabs --blacklist 192.168.1.0/24

To block multiple combinations of hosts and networks use commas

    iptabs --blacklist 192.168.1.2/32,192.168.2.0/24,10.0.0.0/8

The black list may be stored in a file, one per line, in CIDR notation as above. The file may have blank lines or comments. Comments are any line starting with a pound sign.

    iptabs --blacklist-file blacklist.txt

This is hardly practical for spam blocking, but it is useful in other scenarios.

The blacklist chain is dynamic, so it may be modified without recreating the entire firewall. To easily add a host or network

    iptabs --badguy 192.168.1.5/32

The bad guy option can also take multiple hosts and networks separated by commas. This is for use from some monitoring software that detects bad things, such as a root login attempt on a server that does not allow root to login directly.

A file with a list of addresses and/or networks to block can be specified. The format is the same as for the blacklist file option.

    iptabs --badguy-file list.txt

Blacklisting can also use IPSet for faster lookups. This option should be used if a large blacklist table is expected.

    iptabs --use-set --blacklist 192.168.1.1/32

The `--blacklist` option need not be present, using `--use-set` will create an empty set ready to be filled dynamically.

Make sure to specify that IPSet is in use when adding dynamically.

    iptabs --use-set --badguy-file list.txt

Do *NOT* mix IPset and plain blocking. Although this will probably work it will make maintenance difficult.

The IPSet syntax changed not too long ago, so be sure to use the newest version available. For example, Debian 6.0.6 does not have an up to date version, and the kernel module must be built by hand.

Saving IPSet chains persistently does not appear to be present in a lot of distros. On Debian or Ubuntu based systems try the included iptables-persistent file.

DOS/DDOS Protection
-------------------

Not exactly protection, but it might help. Enable with

    iptabs --throttle 20

The number used is the limit per minute for new connections. The burst rate for new connections is four times the specified limit per minute.

    iptabs --est-throttle 40

The number used is the limit per second for established connections. The burst rate for established connections is two times the specified limit per second.

OUTPUT Chain Processing
-----------------------

Normally the OUTPUT chain is set to send all packets, and this should be the default for a desktop or laptop. In the case of a server the OUTPUT chain probably should be filtered.

*Great care should be used with this option.* The trusted interface mechanism will apply to the OUTPUT chain as well. The chain will also respect the blacklist and SSHGuard (if enabled).

    iptabs --output-tcp ssh,http,443,8000-8999 --enable-udp 53

This will trigger output filtering, enable outgoing connections to SSH, HTTP, HTTPS, and the range 8000 through 8999. Also allowing DNS lookups through UDP port 53 (note that the inbound DNS is not enabled, therefoe DNS will fail).

    iptabs --enable-tcp ssh --trusted-iface eth1 --outforce

This will configure a typical "jump server" that can receive SSH, but allow anything in or out of eth1. Note that attempting to SSH out on any other interface (eth0) will fail. the `--outforce` option will trigger OUTPUT chain processing without enabling any outbound services.

The `--outspec` option will specify rules just like --by-spec for the output chain.

Load Balancing Cluster Support
------------------------------

Rules for cluster support can be added.

The minimum specification consists of a virtual IP, number of nodes in the cluster, local node number, and the interface:

    iptabs --interface eth0 --cluster-ip 10.1.1.10 --cluster-nodes 10 --cluster-local 3 --cluster http

The --cluster option must be last. It takes a list of ports or services separated by commas. Ranges are not allowed.

Optionally the virtual MAC and hash type may be specified:

    iptabs --interface eth0 --cluster-ip 10.1.1.10 --cluster-nodes 10 --cluster-local 3 --cluster-mac aa:bb:cc:dd:ee --cluster-hashmode sourceip-sourceport --cluster http

The option `--cluster-mac` defaults to `01:02:03:04:05:06` and the option `--cluster-hashmode` defaults to `sourceip`


Multiple cluster specifications are allowed, but this is not recommended.

    iptabs --interface eth0 --cluster-ip 10.1.1.10 --cluster-nodes 10 --cluster-local 3 --cluster http,https --interface eth1 --cluster-ip 10.1.2.10 --cluster-nodes 10 --cluster-local 3 --cluster-mac aa:bb:cc:dd:ee --cluster http,https

This is far easier using `--file` (see below for details)

    # Clustering web server
    interface eth0
    enable-tcp http,https
    cluster-ip 10.1.1.10
    cluster-nodes 10
    cluster-local 3
    cluster http,https

IPv6 addresses are acceptable with `--cluster-ip` but this is not well tested.

**CAVEATS**

* The service list specified must also be allowed by the firewall itself. This is probably best done **after** the interface specification. See the example above.
* Neither failover nor recovery is triggered at the firewall level, The virtual IP address must be assigned to the device (typically as an alias), this is not done in the code.
* No state information is balanced, this must be done by the application layer.
* Do not use hostnames instead of IP addresses for `--cluster-ip`

Firewall Files
--------------

The firewall tool can read firewall commands from a file instead of the command line. Commands can appear with or without the leading dashes (the command line can do this too, try it!). Comments and blank lines are ignored. A comment is a line that starts with a pound sign.

Here is an example firewall file:

    # Web server firewall
    sshguard
    enable-tcp ssh,http,https
    blacklist 172.168.0.0/16,10.0.0.0/8

This is equivalent to the command line

    iptabs --sshguard --enable-tcp ssh,http,https --blacklist 172.168.0.0/16,10.0.0.0/8

Enjoy!

- - -

About
-----

This is a rewrite in ruby. The original perl script can be found at http://github.com/john-nanney/iptabs
