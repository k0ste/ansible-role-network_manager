# network_manager

Role for deploy profiles for profile-based NetworkManager. Currently works in
`networks-scripts` files format via `ifcfg-rh` NetworkManager plugin

## Requirements for usage

* Ansible 3.0.0+
* ppp, if pppoe is used

## Link types supported

* ethernet
* vlan (802.1q)
* pppoe
* bond
* tunnel (ipip, gre, ?tuntap?)
* bridge

## Example configuration

```yaml
---
network_manager:
  - enable: 'true'
    restart: 'true'
    drop_exists: 'true'
    settings:
      - main:
# Lists system settings plugin names separated by ','. These plugins are used
# to read and write system-wide connection profiles. When multiple plugins are
# specified, the connections are read from all listed plugins. When writing
# connections, the plugins will be asked to save the connection in the order
# listed here; if the first plugin cannot write out that connection type (or
# can't write out any connections) the next plugin is tried, etc. If none of
# the plugins can save the connection, an error is returned to the user.
# The default value and the number of available plugins is distro-specific. See
# the section called “PLUGINS” below for the available plugins. Note that
# NetworkManager's native keyfile plugin is always appended to the end of this
# list (if it doesn't already appear earlier in the list)
        - plugins:
            - action: 'add'
              name: 'ifcfg-rh'
            - action: 'remove'
              name: 'ibft'
# Whether the system uses PolicyKit for authorization. If 'true', non-root
# requests are authorized using PolicyKit. Requests from root (user ID zero) are
# always granted without asking PolicyKit. If 'false', all requests will be
# allowed and PolicyKit is not used. If set to root-only PolicyKit is not used
# and all requests except root are denied. The default value is 'true'
          auth_polkit: 'true'
# This key sets up what DHCP client NetworkManager will use. Allowed values are
# dhclient, dhcpcd, and 'internal'. The 'dhclient' and 'dhcpcd' options require
# the indicated clients to be installed. The 'internal' option uses a built-in
# DHCP client which is not currently as featureful as the external clients. If
# this key is missing, it defaults to 'internal'. If the chosen plugin is not
# available, clients are looked for in this order: 'dhclient', 'dhcpcd',
# 'internal'
          dhcp: 'internal'
# Specify devices for which NetworkManager shouldn't create default wired
# connection (Auto eth0). By default, NetworkManager creates a temporary wired
# connection for any Ethernet device that is managed and doesn't have a
# connection configured. List a device in this option to inhibit creating the
# default connection for the device. May have the special value * to apply to
# all devices. When the default wired connection is deleted or saved to a new
# persistent connection by a plugin, the device is added to a list in the file
# /var/lib/NetworkManager/no-auto-default.state to prevent creating the default
# connection for that device again.
          no_auto_default: "*"
# Specify devices for which NetworkManager will try to generate a connection
# based on initial configuration when the device only has an IPv6 link-local
# address
          assume_ipv6ll_only: ""
# Set the management mode of the hostname. This parameter will affect only the
# transient hostname. If a valid static hostname is set, NetworkManager will
# skip the update of the hostname despite the value of this option. An hostname
# empty or equal to 'localhost', 'localhost6', 'localhost.localdomain' or
# 'localhost6.localdomain' is considered invalid.
# * default: NetworkManager will update the hostname with the one provided via
# DHCP or reverse DNS lookup of the IP address on the connection with the
# default route or on any connection with the property
# hostname.only-from-default set to 'false'. Connections are considered in order
# of increasing value of the hostname.priority property. In case multiple
# connections have the same priority, connections activated earlier are
# considered first. If no hostname can be determined in such way, the hostname
# will be updated to the last one set outside NetworkManager or to
# 'localhost.localdomain'.
# * dhcp: this is similar to 'default', with the difference that after trying
# to get the DHCP hostname, reverse DNS lookup is not done. Note that
# selecting this option is equivalent to setting the property
# 'hostname.from-dns-lookup' to 'false' globally for all connections in
# NetworkManager.conf
# * none: NetworkManager will not manage the transient hostname and will
# never set it
          hostname_mode: 'none'
# Set the DNS processing mode. If the key is unspecified, 'default' is used,
# unless '/etc/resolv.conf' is a symlink to
# '/run/systemd/resolve/stub-resolv.conf', '/run/systemd/resolve/resolv.conf',
# '/lib/systemd/resolv.conf' or '/usr/lib/systemd/resolv.conf'. In that case,
# systemd-resolved is chosen automatically.
# * default: NetworkManager will update '/etc/resolv.conf' to reflect the
# nameservers provided by currently active connections. The rc-manager setting
# (below) controls how this is done.
# * dnsmasq: NetworkManager will run dnsmasq as a local caching nameserver,
# using "Conditional Forwarding" if you are connected to a VPN, and then update
# resolv.conf to point to the local nameserver. It is possible to pass custom
# options to the dnsmasq instance by adding them to files in the
# "/etc/NetworkManager/dnsmasq.d/" directory. Note that when multiple upstream
# servers are available, dnsmasq will initially contact them in parallel and
# then use the fastest to respond, probing again other servers after some time.
# This behavior can be modified passing the 'all-servers' or 'strict-order'
# options to dnsmasq
# * systemd-resolved: NetworkManager will push the DNS configuration to
# systemd-resolved
# * none: NetworkManager will not modify resolv.conf. This implies 'rc_manager'
# unmanaged
# Note that the plugins 'dnsmasq' and 'systemd-resolved' are caching local
# nameservers. Hence, when NetworkManager writes
# '/var/run/NetworkManager/resolv.conf' and '/etc/resolv.conf' (according to
# 'rc_manager' setting), the name server there will be localhost only.
# NetworkManager also writes a file
# '/var/run/NetworkManager/no-stub-resolv.conf' that contains the original name
# servers pushed to the DNS plugin. When using 'dnsmasq' and 'systemd-resolved'
# per-connection added dns servers will always be queried using the device the
# connection has been activated on
          dns: 'default'
# Set the resolv.conf management mode. This option is about how NetworkManager
# writes to /etc/resolv.conf, if at all. The default value depends on
# NetworkManager build options, and this version of NetworkManager was build
# with a default of "symlink". Regardless of this setting, NetworkManager will
# always write its version of resolv.conf to its runtime state directory as
# '/var/run/NetworkManager/resolv.conf'. If you configure dns='none' or make
# '/etc/resolv.conf' immutable, NetworkManager will ignore this setting and
# always choose 'unmanaged'
# * auto: if systemd-resolved plugin is configured via the dns setting or if it
# gets detected as main DNS plugin, NetworkManager will update systemd-resolved
# without touching '/etc/resolv.conf'. Alternatively, if 'resolvconf' or
# 'netconfig' are enabled at compile time and the respective binary is found,
# NetworkManager will automatically use it. Note that if you install or
# uninstall these binaries, you need to reload the 'rc_manager' setting with
# SIGHUP or systemctl reload NetworkManager. As last fallback it uses the
# 'symlink' option
# * symlink: If '/etc/resolv.conf' is a regular file or does not exist,
# NetworkManager will write the file directly. If '/etc/resolv.conf' is instead
# a symlink, NetworkManager will leave it alone. Unless the symlink points to
# the internal file '/var/run/NetworkManager/resolv.conf', in which case the
# symlink will be updated to emit an inotify notification. This allows the user
# to conveniently instruct NetworkManager not to manage '/etc/resolv.conf' by
# replacing it with a symlink
# * file: NetworkManager will write '/etc/resolv.conf' as regular file. If it
# finds a symlink to an existing target, it will follow the symlink and update
# the target instead. In no case will an existing symlink be replaced by a file.
# * resolvconf: NetworkManager will run `resolvconf`` to update the DNS
# configuration
# * netconfig: NetworkManager will run netconfig to update the DNS configuration
# * unmanaged: don't touch '/etc/resolv.conf'
          rc_manager: 'auto'
# Send the connection DNS configuration to systemd-resolved. Defaults to "true"
# Note that this setting is complementary to the dns setting. You can keep this
# enabled while using dns set to another DNS plugin alongside systemd-resolved,
# or dns set to systemd-resolved to configure the system resolver to use
# systemd-resolved. If systemd-resolved is enabled, the connectivity check
# resolves the hostname per-device
          systemd_resolved: 'true'
# Comma separated list of options to aid debugging. This value will be combined
# with the environment variable NM_DEBUG. Currently, the following values are
# supported:
# * RLIMIT_CORE: set 'ulimit -c unlimited' to write out core dumps. Beware,
# that a core dump can contain sensitive information such as passwords or
# configuration settings.
# * fatal-warnings: set g_log_set_always_fatal() to core dump on warning
# messages from glib. This is equivalent to the '--g-fatal-warnings' command
# line option
          debug:
            - 'RLIMIT_CORE'
            - 'fatal-warnings'
# The number of times a connection activation should be automatically tried
# before switching to another one. This value applies only to connections that
# can auto-connect and have a connection.autoconnect-retries property set to
# -1. If not specified, connections will be tried 4 times. Setting this value
# to 1 means to try activation once, without retry.
          autoconnect_retries_default: '1'
# This key specifies in which order slave connections are auto-activated on
# boot or when the master activates them. Allowed values are name (order
# connection by interface name, the default), or index (order slaves by their
# kernel index).
          slaves_order: ''
# The firewall backend for configuring masquerading with shared mode. Set to
# either 'iptables', 'nftables' or 'none'. 'iptables' and 'nftables' require
# 'iptables' and 'nft' application, respectively. 'none' means to skip firewall
# configuration if the users wish to manage firewall themselves. If
# unspecified, it will be auto detected.
          firewall_backend: 'none'
# If the value is 'auto' (the default), IWD is queried for its current state
# directory when it appears on D-Bus - the directory where IWD keeps its network
# configuration files - usually '/var/lib/iwd'. NetworkManager will then attempt
# to write copies of new or modified Wi-Fi connection profiles, converted into
# the IWD format, into this directory thus making IWD connection properties
# editable. NM will overwrite existing files without preserving their contents.
# The path can also be overriden by pointing to a specific existing and
# writable directory. On the other hand setting this to an empty string or any
# other value disables the profile conversion mechanism. This mechanism allows
# editing connection profile settings such as the 802.1x configuration using
# NetworkManager clients. Without it such changes have no effect in IWD
          iwd_config_path: '/var/lib/iwd'
        logging:
# The default logging verbosity level. One of 'OFF', 'ERR', 'WARN', 'INFO',
# 'DEBUG', 'TRACE', in order of verbosity. 'OFF' disables all logging. 'INFO'
# is the default verbosity for regular operation. 'TRACE' is for debugging. The
# other levels are in most cases not useful. For example, 'DEBUG' is between
# 'TRACE' and 'INFO', but it's too verbose for regular operation and lacks
# possibly interesting messages for debugging. Almost always, when debugging an
# issue or reporting a bug, collect full level 'TRACE' logs to get the full
# picture
          - level: 'INFO'
# Filter the messages by their topic. When debugging an issue, it's better to
# collect all logs (ALL domain) upfront. The unnecessary parts can always be
# ignored later. In the uncommon case to tune out certain topics, the following
# log domains are available: 'PLATFORM', 'RFKILL', 'ETHER', 'WIFI', 'BT', 'MB',
# 'DHCP4', 'DHCP6', 'PPP', 'WIFI_SCAN', 'IP4', 'IP6', 'AUTOIP4', 'DNS', 'VPN',
# 'SHARING', 'SUPPLICANT', 'AGENTS', 'SETTINGS', 'SUSPEND', 'CORE', 'DEVICE',
# 'OLPC', 'WIMAX', 'INFINIBAND', 'FIREWALL', 'ADSL', 'BOND', 'VLAN', 'BRIDGE',
# 'DBUS_PROPS', 'TEAM', 'CONCHECK', 'DCB', 'DISPATCH', 'AUDIT', 'SYSTEMD',
# 'VPN_PLUGIN', 'PROXY'. In addition, these special domains can be used: 'NONE',
# 'ALL', 'DEFAULT', 'DHCP', 'IP'
            domains: 'DEFAULT'
# The logging backend. Supported values are 'syslog' and 'journal'. When
# NetworkManager is started with "--debug" in addition all messages will be
# printed to stderr. If unspecified, the default is "journal"
            backend: 'journal'
# Whether the audit records are delivered to auditd, the audit daemon. If
# 'false', audit records will be sent only to the NetworkManager logging system.
# If set to 'true', they will be also sent to auditd. The default value is
# 'true'
            audit: 'true'
    interfaces:
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
# Default MTU for this device
      mtu: '9000'
# List of options to the bonding driver for this interface.
      bonding_options:
        mode: '4'
        miimon: '100'
        xmit_hash_policy: '2'
        lacp_rate: '1'
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
      bootproto: 'none'
# List of bridging options for either the bridge device, or the port device
      bridging_opts:
        hello_time: '200'
        priority: '65535'
    - description: 'Bonding Slave 0'
      interface: 'eth0'
      type: 'Ethernet'
      onboot: 'yes'
# Enable LLDP stack support or not
      lldp: 'yes'
      bootproto: 'none'
# Specifies device as a slave.
      slave: 'yes'
# Specifies master device to bind to.
      master: 'bond0'
  # Task will be executed when interface go to the 'up' state
      ifup_local:
        - 'ip link set dev eth0 txqueuelen 10000'
        - 'ethtool -K eth0 gso off tso off gro off tx off rx off sg off'
    - description: 'Bonding Slave 1'
      interface: 'eth1'
      type: 'Ethernet'
      onboot: 'yes'
      bootproto: 'none'
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
# the specified bridge device instead
      bridge: 'br1'
# Time that the system should pause after the specific interface is enabled.
# This may be useful if one interface is connected to a switch which has
# spanning tree enabled and must wait for STP to converge before the interface
# should be considered usable
      link_delay: '3'
# If 'yes' adds 'arp' flag to ip, for use with the ethertap device
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
# presence of a link before starting dhclient
      dhcp_persistent: 'yes'
# With this option set to 'yes', when a dhcp configured interface is brought
# down with 'ifdown', the lease will be released. Otherwise, leases are not
# released
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
      tun_local: '5.128.220.250'
      tun_remote: '5.128.220.150'
      address_local: '192.168.168.1'
      address_remote: '192.168.168.2'
    - interface: 'Tunnel1'
      type: 'GRE'
      onboot: 'yes'
      bootproto: 'none'
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
    ifup_global: |
      function main() {
        if ip link set dev "${1}" txqueuelen 10000
      }
```
