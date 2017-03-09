PeerVPN
=======

This Ansible role is used in [Kubernetes the not so hard way with Ansible (at Scaleway) - Part 3 - PeerVPN](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-3/). Used to setup [PeerVPN](https://peervpn.net/) for Ubuntu 16.04 (but should basically work with all Linux OS that use systemd). With PeerVPN you can easily setup a fully meshed VPN across datacenter and all nodes you like. You only need at least one host with a public reachable interface (default is Port 7000 protocol UDP). One simple configration could be that you use this public reachable host for your `peervpn_conf_initpeers` setting. Finding the other hosts on your VPN will be automagically done by PeerVPN.

PeerVPN installes it's own TAP interface for it's purpose. The default name of that TAP interface is `tap0`. To change the name specify a different value for `peervpn_conf_interface` variable.

To generate a strong secret password for your PeerVPN preshared key you can use:

```
openssl rand -base64 382 | tr -d '\n' && echo
```
Since it's a preshared key this key MUST be used on all hosts where you install PeerVPN and use the same network name. Otherwise connection won't work. The default preshared key is `default` which you want to change of course ;-)

Requirements
------------

Allow traffic on port 7000 protocol UDP (default) if you have firewall rules installed. You also NEED to add `peervpn_conf_initpeers` variable. There is no default for this variable! IPv6 is ENABLED by default. If you don't want to use it add a variable `peervpn_conf_enableipv6: no`.

Role Variables
--------------

Basically you only need to change very few variables (see below). But have a look at `templates/etc/peervpn/peervpn.conf.j2` for examples and full description of the variables.

Variables with NO default values:
```
peervpn_conf_initpeers
peervpn_conf_engine
peervpn_conf_ifconfig6
peervpn_conf_upcmd
peervpn_conf_chroot
```
Variables with default values:

```
peervpn_version: "peervpn-0-044"
peervpn_install_directory: "/opt/{{peervpn_version}}"
peervpn_dest: "/usr/local/sbin"
peervpn_conf_networkname: "peervpn"
peervpn_conf_psk: "default"
peervpn_conf_enabletunneling: "yes"
peervpn_conf_interface: "tap0"
peervpn_conf_local: "0.0.0.0"
peervpn_conf_port: 7000
peervpn_conf_ifconfig4: "10.0.0.1/24"
peervpn_conf_sockmark: 0
peervpn_conf_enableipv4: "yes"
peervpn_conf_enablenat64clat: "no"
peervpn_conf_enablerelay: "no"
peervpn_conf_enableprivdrop: "yes"
peervpn_conf_user: "nobody"
peervpn_conf_group: "nogroup"
```

You MUST specify a value for `peervpn_conf_initpeers` to make any use of PeerVPN (either per host in Ansible `host_vars` directory or per host group in `group_vars` directory. E.g. if you specify `peervpn_conf_initpeers: "host.example.net 7000"` PeerVPN tries to connect to `host.example.net` on port `7000` via UDP to setup a connection.

You should at least change the following variables:

`peervpn_conf_initpeers`: The hostname and port PeerVPN should connect to become part of the VPN.
`peervpn_conf_networkname`: The name of your VPN.
`peervpn_conf_psk`: Preshared key. How to generate a good preshared key password see introduction above.
`peervpn_conf_ifconfig4`: The IP address of the node and subnet in CIDR notation. This variables needs to be specified per host of course.

Example Playbook
----------------

```
- hosts: webservers
  roles:
    - peervpn
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
