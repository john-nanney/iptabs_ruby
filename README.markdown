
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

Configuration is a single command line invocation. With no arguments, the firewall will allow incoming SSH connections with IPv4 and IPv6, denying all other incoming connections.

To display a help message:

	iptabs --help

To allow SSH over IPv4 and IPv6:

	iptabs

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

DOS/DDOS Protection
-------------------

Not exactly protection, but it might help. Enable with

    iptabs --throttle 20

The number used is the limit per minute for new connections. The burst rate for new connections is four times the specified limit per minute.

    iptabs --est-throttle 40

The number used is the limit per second for established connections. The burst rate for established connections is two times the specified limit per second.

Enjoy!

About
-----

This is a rewrite in ruby. The original perl script can be found at http://github.com/john-nanney/iptabs
