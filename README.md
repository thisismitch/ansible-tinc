# Introduction

The primary purpose of this playbook is to secure the private network of DigitalOcean Droplets with tinc VPN. You can use it with any Ubuntu servers that can reach each other over a network.

This sets up a tinc VPN between several servers. It also adds /etc/hosts entries for the inventory hostnames to resolve to the VPN IP addresses.

## Prerequisites

This playbook been tested on Ubuntu 14.04 and CentOS 7 servers.

Your local machine (where Ansible is installed) must be able to log in to the remote servers as "root", preferably with passwordless public SSH key, which is specified as the `remote_user` in `/ansible.cfg`. Due to a [bug with the Ansible Synchronize module](https://github.com/ansible/ansible/issues/13825), it is not possible to use a different `remote_user` at this time.

By default, this playbook will bind tinc to the IP address on the `eth1` interface (private network interface on DigitalOcean Droplets). See the "Review Group Variables" section to change this.

## Preparation

### Create Inventory

Create a `/hosts` file with the nodes that you want to include in the VPN:

```
prod01 vpn_ip=10.0.0.1 ansible_host=162.243.125.98
prod02 vpn_ip=10.0.0.2 ansible_host=162.243.243.235
prod03 vpn_ip=10.0.0.3 ansible_host=162.243.249.86
prod04 vpn_ip=10.0.0.4 ansible_host=162.243.252.151
```

- The first column is the inventory hostname, "prod01" in the example, how ansible will refer to the host
- `vpn_ip` is the IP address that the node will use for the VPN
- `ansible_host` must be set to a value that your ansible machine can reach the node at

**Note:** The inventory hostname, which we are using as each node's name in Tinc, can't contain characters that Tinc doesn't allow for node names. For example, hyphens (`-`) are not allowed.

### Review Group Variables

The `/group_vars/all` file contains a few values that you may want to modify.

- `physical_ip` specifies which IP address you want tinc to bind to, based on network interface name. It is set to `eth1` (ansible_eth1) by default. On DigitalOcean, `eth1` is the private network interface so *Private Networking* must be enabled unless you would rather use the public network interface (`eth0`)
- `netname` specifies the tinc netname. It's set to `nyc3` by default.
- `vpn_netmask` specifies the netmask that the will be applied to the VPN interface. By default, it's set to `255.255.255.0`, which means that each `vpn_ip` is a Class C address which can only communicate with other hosts within the same subnet. For example, a `10.0.0.x` will not be able to communicate with a `10.0.1.x` host unless the subnet is enlarged by changing `vpn_netmask` to something like `255.255.0.0`.

The other variables probably don't need to be modified.

- `vpn_interface` is the virtual network interface that tinc will use. It is `tun0` by default.

## Set Up Tinc

Run the playbook:

```bash
ansible-playbook site.yml
```

After the playbook completes, all of the hosts in the inventory file should be able to communicate with each other over the VPN network.

## Quick Test

Log in to your first host and ping the second host:

```bash
ping 10.0.0.2
```

Or, assuming one of your hosts is named `prod02`, run this:

```bash
ping prod02
```

Feel free to test the other nodes.

## Adding and removing hosts

You may add or remove hosts at any time by updating the hosts file and running the playbook. Note that removed hosts will result in orphaned tinc hosts files and `/etc/hosts` entries.

## Running Multiple VPNs

This playbook does not support multiple VPNs but it could be easily extended.
