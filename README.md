# Tinc

This sets up a tinc vpn between several servers.

## Running

Create a hosts file.

```bash
$ cat hosts
trusty1 ansible_ssh_host=x.y.z.21
trusty2 ansible_ssh_host=x.y.z.22
trusty3 ansible_ssh_host=x.y.z.23
trusty4 ansible_ssh_host=x.y.z.24
```

Next up, run ansible.

```bash
$ ansible-playbook site.yml
```

If that completes OK, you should be able to ping all the hosts from all the other hosts on the vpn network.

```bash
ubuntu@trusty1:~$ ip ro sh
default via 104.193.173.17 dev eth0
10.0.0.0/24 dev tun0  proto kernel  scope link  src 10.0.0.21
104.193.173.16/28 dev eth0  proto kernel  scope link  src 104.193.173.21
ubuntu@trusty1:~$ ping -c 1 -w 1 10.0.0.23
PING 10.0.0.23 (10.0.0.23) 56(84) bytes of data.
64 bytes from 10.0.0.23: icmp_seq=1 ttl=64 time=0.751 ms

--- 10.0.0.23 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.751/0.751/0.751/0.000 ms
ubuntu@trusty1:~$ ping -c 1 -w 1 10.0.0.24
PING 10.0.0.24 (10.0.0.24) 56(84) bytes of data.
64 bytes from 10.0.0.24: icmp_seq=1 ttl=64 time=1.34 ms

--- 10.0.0.24 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.348/1.348/1.348/0.000 ms

```

## ip addressing

There is an ansible custom facts script that creates a vpn ip address based on the last octet of the ip address reported by ```hostname -i``` and that is added to 10.0.0. So, of course this won't work in some situations, eg. if you have two servers with the same last octet. But I didn't want to configure a ```host_vars``` entry for each server (which is the other option). Otherwise I guess you'd need some kind of ip management, such as dhcp running on one of the servers? Man, what a pain ips are. lol.

So in the example below the eth0 ipv4 address is x.y.z.23 so the ```vpn_ip``` becomes 10.0.0.21.

```bash
$ ansible -m setup trusty1 | grep -A 4 "ansible_local\|ansible_eth0"
"ansible_eth0": {
  "active": true,
  "device": "eth0",
  "ipv4": {
    "address": "x.y.z.21",
    --
    "ansible_local": {
      "tinc_facts": {
        "vpn_ip": "10.0.0.21"
      }
    },

```

## Network

```bash
$ ip route show
default via x.y.z.17 dev eth0
10.0.0.0/24 dev tun0  proto kernel  scope link  src 10.0.0.22
x.y.z.16/28 dev eth0  proto kernel  scope link  src x.y.z.22
```

## Running iperf

```bash
ubuntu@trusty2:~$ iperf -s 10.0.0.22
iperf: ignoring extra argument -- 10.0.0.22
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.0.0.22 port 5001 connected with 10.0.0.21 port 35705
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.1 sec   312 MBytes   260 Mbits/sec
```

```bash
ubuntu@trusty1:~$ iperf -c 10.0.0.22
------------------------------------------------------------
Client connecting to 10.0.0.22, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.0.0.21 port 35705 connected with 10.0.0.22 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec   312 MBytes   262 Mbits/sec
```

## Todo

* Use IPv6 instead?
