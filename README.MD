ansible-role-network-scripts
==============================

Role for deploy profiles for RHEL profile-based network manager.

Requirements for usage
-----------------------

* Ansible 2.2;
* jmespath;
* [python-netaddr](//docs.ansible.com/ansible/playbooks_filters_ipaddr.html);
* ppp, if pppoe is used;

Link types supported
-----------------------

* ethernet (inc. pseudo, like [IMQ](//github.com/imq/linuximq));
* vlan (802.1q);

Extra
-----------

* ip addresses, before deploy will be checked by Jinja with python-netaddr lib;
* Support 'static' and 'dhcp' connections;
* Support arrays of addresses;
* Support assign options for Bonding Linux module;
* Support PostUp and PreDown command execution;

Example configuration
-------------------------

```yaml
---
network_scripts_interfaces:
- description: 'Bonding master'
  interface: 'bond0'
  type: 'Bond'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  mtu: '9000'
  bonding_master: 'yes'
  bonding_options:
    mode: '4'
    miimon: '100'
    xmit_hash_policy: '2'
    lacp_rate: '1'
- description: 'Bonding Slave'
  interface: 'eth0'
  type: 'Ethernet'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  slave: 'yes'
  master: 'bond0'
- description: 'Bonding Slave'
  interface: 'eth1'
  type: 'Ethernet'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  slave: 'yes'
  master: 'bond0'
- description: 'VLAN 2'
  interface: 'vlan2'
  type: 'Ethernet'
  onboot: 'yes'
  bootproto: 'none'
  nm_controlled: 'no'
  vlan: 'yes'
  mtu: '9000'
  physdev: 'bond0'
  macaddr: '10:c3:7b:4f:58:a4'
  address: '82.200.50.62/30'
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
  tun_local: '5.128.220.250'
  tun_remote: '5.128.220.150'
  address_local: '192.168.168.1'
  address_remote: '192.168.168.2'
  mtu: '1476'
  ttl: '255'
```
