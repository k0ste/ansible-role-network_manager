# network-scripts

Role for deploy profiles for RHEL profile-based network manager.

## Requirements for usage

* Ansible 3.0.0+;
* ppp, if pppoe is used;

## Link types supported

* ethernet (inc. pseudo, like [IMQ](//github.com/imq/linuximq));
* vlan (802.1q);
* pppoe;
* bond;
* tunnel (ipip, gre, ?tuntap?);
* bridge;

## Extra

* ip/mac addresses, before deploy will be checked by Jinja with python-netaddr
lib;
* Support 'static' and 'dhcp' connections;
* Support arrays of addresses;
* Support assign options for `bonding` devices;
* Support assign options for `bridging` devices;
* Support ifdown and ifup triggers (for network-scripts);
* Support NetworkManager dispatcher.d triggers (for NetworkManager)

## Example configuration

```yaml
---
# Enable 'network' service.
network_scripts_enable: 'true'
# Restart 'network' service. May be dangerous!
network_scripts_restart: 'false'
# Deploy global settings.
network_scripts_deploy_global: 'true'
# Deploy interfaces or not. This can be used when you need triggers but do not
# need interfaces. For example, because the interfaces are controlled by a
# different entity (i.e. OpenStack, oVirt or Proxmox).
network_scripts_deploy_interfaces: 'true'
# Deploy triggers - custom commands in the scripts /sbin/ifup-local,
# ifdown-pre-local, and ifdown-local. If you modified the initscripts themselves
# (for example '/etc/sysconfig/network-scripts/ifup-eth') then those changes
# would be overwritten by an initscripts package update. Therefore it is
# recommend that you avoid modifying the initscripts directly and make use of
# the /sbin/if*local scripts, so that your custom changes will survive package
# updates. The initscripts just check for the presence of the relevant
# /sbin/if*local and run them if they exist. The initscripts do not place
# anything in the /sbin/if*local scripts, nor does the initscripts RPM (or any
# package) own or modify those files.
network_scripts_deploy_triggers: 'true'
# Type of triggers for deploy:
# 'ifcfg' - for network-scripts
# 'dispatcher' - for NetworkManager
network_scripts_triggers_type: 'dispatcher'
# Delete exists interface profiles. If 'true' - all files in
# /etc/sysconfig/network-scripts matched with regex '^ifcfg-(.*)' will be
# deleted.
network_scripts_drop_exists: 'true'
# Deploy PAP & CHAP ppp secrets, or not.
network_scripts_ppp_secrets: 'true'
# The global functions for ifup_local. This content will be putted to
# ifup_local script and functions can be accessed for any interface via
# - 'main "<iface_name>"'
network_scripts_ifup_global: |
  function main() {
    if ip link set dev "${1}" txqueuelen 10000
  }

network_scripts_global:
# Enable or disable global forwarding of incoming IPv6 packets on all interfaces.
# Actual packet forwarding cannot be controlled per-device, use netfilter6 for
# such issues.
- ipv6_forwarding: 'no'
# Sets the default for device-based autoconfiguration.
# Default: 'yes' if ipv6_forwarding: 'no', 'no' if ipv6_forwarding: 'yes'.
  ipv6_autoconf: 'yes'
# Sets the default for device-based Host/Router behaviour.
# Default: 'yes' if ipv6_forwarding: 'yes', 'no' if ipv6_forwarding: 'no'.
  ipv6_router: 'no'
# Controls automatic IPv6 tunneling. Default is 'no'.
  ipv6_auto_tunnel: 'no'
  no_zeroconf: 'yes'
# Delay in seconds after all network interfaces are initialized. Useful if
# network has spanning tree running and must wait for STP convergence.
  network_delay: '0'

network_scripts_interfaces:
# Friendly name for users to see. Most important for PPP. Only used in front
# ends.
- description: 'Bonding master'
# Name of physical device (except dynamically-allocated PPP devices where it is
# the "logical name").
  interface: 'bond0'
  type: 'Bond'
  onboot: 'yes'
# 'bootp' or 'dhcp' cause a DHCP client to run on the device. Any other value
# causes any static configuration in the file to be applied.
  bootproto: 'none'
# If set to 'no', NetworkManager will ignore this connection/device.
# Defaults to 'yes'.
  nm_controlled: 'no'
# Default MTU for this device
  mtu: '9000'
# List of options to the bonding driver for this interface.
  bonding_options:
    mode: '4'
    miimon: '100'
    xmit_hash_policy: '2'
    lacp_rate: '1'
- interface: 'imq0'
  onboot: 'yes'
  nm_controlled: 'no'
  bootproto: 'none'
- interface: 'br1'
  type: 'Bridge'
# Sets the bridge's 'bridge forward delay' (in seconds).
  delay: '0'
# Controls this bridge instance's participation in the spanning tree protocol.
# If 'on' the STP will be turned on, otherwise it will be turned off. When
# turned off, the bridge will not send or receive BPDUs, and will thus not
# participate in the spanning tree protocol. If your bridge isn't the only
# bridge on the LAN, or if there are loops in the LAN's topology, DO NOT turn
# this option off. If you turn this option off, please know what you are doing.
  stp: 'off'
  onboot: 'yes'
  nm_controlled: 'no'
  bootproto: 'none'
# List of bridging options for either the bridge device, or the port device.
  bridging_opts:
    hello_time: '200'
    priority: '65535'
- description: 'Bonding Slave 0'
  interface: 'eth0'
  type: 'Ethernet'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
# Specifies device as a slave.
  slave: 'yes'
# Specifies master device to bind to.
  master: 'bond0'
  # Task executed when interface comes up
  ifup_local:
  - 'ip link set dev eth0 txqueuelen 10000'
  - 'ethtool -K eth0 gso off tso off gro off tx off rx off sg off'
- description: 'Bonding Slave 1'
  interface: 'eth1'
  type: 'Ethernet'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  slave: 'yes'
  master: 'bond0'
  ifup_local:
  - 'ip link set dev eth1 txqueuelen 10000'
  - 'ethtool -K eth1 gso off tso off gro off tx off rx off sg off'
- description: 'VLAN2'
  interface: 'vlan2'
  type: 'Vlan'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  vlan: 'yes'
  mtu: '9000'
  physdev: 'bond0'
  physaddr: '9c:b6:d0:95:a7:1d'
  macaddr: '10:c3:7b:4f:58:a4'
  address: '82.200.50.62/30'
  # You can use "$1" as interface name
  ifup_local:
  - 'ip route add 0.0.0.0/0 via 82.200.50.61 dev $1 table table100'
  - 'ip route add 0.0.0.0/0 via 82.200.50.61 dev $1 table table200'
  - 'ip route add 0.0.0.0/0 via 82.200.50.61 dev $1 table table300'
  - 'ip rule add from 82.200.50.62 lookup table100'
  - 'ip rule add from all fwmark 200 lookup table200'
  - 'ip rule add from all fwmark 300 lookup table300'
  # Task executed when interface goes down
  ifdown_local:
  - 'ip rule delete from 82.200.50.62 lookup table100'
  - 'ip rule delete from all fwmark 200 lookup table200'
  - 'ip rule delete from all fwmark 300 lookup table300'
  - 'ip route delete 0.0.0.0/0 via 82.200.50.61 dev $1 table table100'
  - 'ip route delete 0.0.0.0/0 via 82.200.50.61 dev $1 table table200'
  - 'ip route delete 0.0.0.0/0 via 82.200.50.61 dev $1 table table300'
- description: 'VLAN3'
  interface: 'vlan3'
  type: 'Vlan'
  onboot: 'yes'
  bootproto: 'static'
  vlan: 'yes'
  physdev: 'bond0'
  address: '82.200.50.2/30'
  gateway: '82.200.50.1'
# Use the specified source address for outgoing packets.
  src_address: ''
# Metric for the default route using 'gateway'.
  metric: '100'
- description: 'VLAN48'
  interface: 'vlan48'
  type: 'Vlan'
  vlan: 'yes'
# If set, the ethernet device is not assigned an address. It is added to
# the specified bridge device instead.
  bridge: 'br1'
# Time that the system should pause after the specific interface is enabled.
# This may be useful if one interface is connected to a switch which has
# spanning tree enabled and must wait for STP to converge before the interface
# should be considered usable.
  link_delay: '3'
# If 'yes' adds 'arp' flag to ip, for use with the ethertap device.
  arp: 'no'
- description: 'VLAN4'
  interface: 'vlan4'
  type: 'Vlan'
  onboot: 'yes'
  bootproto: 'dhcp'
  vlan: 'yes'
  physdev: 'bond0'
# Without this option, or if it is 'no', and bootproto: 'dhcp', dhclient is run
# for the interface in "one-shot" mode. If the dhcp server does not respond for
# a configurable timeout, then dhclient exits and the interface is not brought
# up - the '-1' option is given to dhclient. If 'yes', then dhclient will keep
# on trying to contact the dhcp server when it does not respond - no '-1' option
# is given to dhclient. Note: this disables the automatic checking for the
# presence of a link before starting dhclient.
  dhcp_persistent: 'yes'
# With this option set to 'yes', when a dhcp configured interface is brought
# down with 'ifdown', the lease will be released. Otherwise, leases are not released.
  dhcp_release: 'yes'
# Tells to not obtain hostname from DHCP server in the ifup-post phase. This
# option might be useful especially with static configuration of the interface.
  no_dhcp_hostname: 'yes'
# Tells initscripts whether the DHCP_HOSTNAME or DHCP_FQDN options (below)
# should be sent to DHCP server.
  dhcp_send_hostname: 'yes'
# Sends the specified hostname to the DHCP server.
  dhcp_hostname: 'el7'
# Sends the specified FQDN to the DHCP server. Please note when both
# 'dhcp_hostname' and 'dhcp_fqdn' are specified, only 'dhcp_fqdn' will be used.
  dhcp_fqdn: 'el7.redhat.com'
# If set to 'yes', it will cause dhclient-script to ignore any $GATEWAY setting
# that may be in the ifcfg file for this interface. Otherwise, the dhclient
# session which obtains an ip-address on the same subnet as $GATEWAY will set
# the default route to be via $GATEWAY, and no other dhclient session will set
# the default route.
  dhcp_client_ignore_gateway: 'yes'
# Any additional arguments to dhclient.
  dhcp_client_args: ''
# If set to 'no', ifup will not try to determine, if requested ip address is
# used by other machine in network. Defaults to 'yes'.
  arp_check: 'no'
# If set to 'no' the neighbours in current network will not be updated with ARP
# information about this NIC. This is especially handy using LVS Load
# Balancing with Direct Routing enabled. Defaults to 'yes'.
  arp_update: 'yes'
# If set to yes, ifup-eth will end immediately after ipv4 dhclient fails.
# Defaults to 'no'.
  ipv4_failure_fatal: 'no'
- interface: 'Tunnel0'
  type: 'IPIP'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  tun_local: '5.128.220.250'
  tun_remote: '5.128.220.150'
  address_local: '192.168.168.1'
  address_remote: '192.168.168.2'
- interface: 'Tunnel1'
  type: 'GRE'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
# IP address of the local tunnel endpoint. If unspecified, an IP address is
# selected automatically for outgoing tunnel packets, and incoming tunnel
# packets are accepted on all local IP addresses.
  tun_local: '5.128.220.250'
# IP address of the remote tunnel endpoint.
  tun_remote: '5.128.220.150'
# Local IP address of the tunnel interface.
  address_local: '192.168.168.1'
# IP address of the remote end of the tunnel interface If this is specified, a
# route to 'address_remote' through the tunnel is added automatically.
  address_remote: '192.168.168.2'
  mtu: '1476'
# TTL value for tunnel packets. Default is to use the TTL of the packet
# transported through the tunnel.
  ttl: '255'
- interface: 'ppp0'
  type: 'xDSL'
  physdev: 'vlan888'
  ppp_user: 'ppp_USERQWFWF'
  ppp_password: 'pspspdq'
# Set this interface as default route. Default is 'yes'.
  defroute: 'no'
# Turns on/off pppd and chat (if used) debugging. Default is 'yes'.
  debug: 'yes'
# Simplified interface here doesn't let people specify which characters to
# escape. Almost everyone can use asyncmap 00000000 anyway, and they can set
# PPPOPTIONS to asyncmap foobar if they want to set options perfectly.
  escapeshars: 'no'
# 'yes' implies "modem crtscts" options.
  hardware_flow_control: 'no'
# "name $PAPNAME" on pppd command line (note that the "remotename" option is
# always specified as the logical ppp device name, like "ppp0" (which might
# perhaps be the physical device ppp1 if some other ppp device was brought up
# earlier), which makes it easy to manage pap/chap files - name/password pairs
# are associated with the logical ppp device name so that they can be managed
# together.
  papname: ''
# Remote ip address, normally unspecified.
  remote_address: ''
# Number of seconds, default is '5'. Time to wait before re-establishing the
# connection after a successfully-connected session terminates before attempting
# to establish a new connection.
  disconnect_timeout: '5'
# Number of seconds, default is '60'. Time to wait before re-attempting to
# establish a connection after a previous attempt fails.
  retry_timeout: '60'
# If this is 'yes' (which is default), then we will re-run pppd if it exits
# with a "connect script failed" status. Otherwise, only one attempt is made to
# bring up the connection. Note that some connect scripts (for example, wvdial)
# might do their own retries (such as BUSY or NO DIALTONE conditions).
  retry_connect: 'yes'
# If this is set, this will cause ppp-watch to exit after the specified number
# of attempts.
  max_fail: ''
# Switches on demand-dialing mode using pppd's "demand" option.
  demand: 'no'
# The amount of time the link needs to be inactive before pppd will bring it
# down automatically. Default is '600'.
  idle_timeout: '600'
# The amount of time to wait at boot before giving up on the connection.
  boot_timeout: '30'
# Modify /etc/resolv.conf if peer uses msdns extension.
  peer_dns: 'no'
# Do not allow to user (not 'root') manage this connection.
  user_ctl: 'no'
# Set mss size, default is 'no'.
  clampmss: 'no'
# Another xDSL options. Set defaults.
  firewall: 'NONE'
  pid_file: '/var/run/pppoe-ppp0.pid'
  pppoe_timeout: '80'
  lcp_failure: '3'
  lcp_interval: '20'
  connect_poll: '6'
  connect_timeout: '60'
```
