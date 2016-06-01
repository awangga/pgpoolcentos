# Bridging network for VM
Setup bridge for VM connection


### setup static ip

disable Network Manager

```sh
# systemctl stop NetworkManager 
# systemctl disable NetworkManager
```
edit /etc/sysconfig/network-scripts/ifcfg-bond0

```sh
DEVICE=bond0
BONDING_OPTS="miimon=1 updelay=0 downdelay=0 mode=balance-rr"
TYPE=Bond
BONDING_MASTER=yes
BOOTPROTO=static
#IPADDR=10.200.0.17
#PREFIX=24
#GATEWAY=10.200.0.1
#DEFROUTE=yes
#IPV4_FAILURE_FATAL=no
#IPV6INIT=no
#NAME=bond0
#UUID=7a971c0b-4e70-405f-ad97-969bd98f653c
ONBOOT=yes
#DNS1=10.200.0.20
BRIDGE=bridge0


```

edit /etc/sysconfig/network-scripts/ifcfg-bridge0

```sh
DEVICE="bridge0"
# BOOTPROTO is up to you
# If you prefer “static”, you will need to
# specify the IP address, netmask,gatewayand DNS information.
BOOTPROTO="static"
#IPV6INIT="yes"
#IPV6_AUTOCONF="yes"
ONBOOT="yes"
TYPE="Bridge"
#DELAY="0"
IPADDR=10.200.0.17
NETMASK=255.255.255.0
#GATEWAY=10.200.0.10
#DNS1=10.200.0.10
```
edit gateway /etc/sysconfig/network

```sh
GATEWAY=10.200.0.1
```

restart network, check bridge, check routing delete other routing bridge if any

```sh
# brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.fe5400871d74	no		vnet0
							vnet1
bridge0		8000.901b0e81e9e0	no		bond0
virbr0		8000.525400ecc5ec	yes		virbr0-nic
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.200.0.10     0.0.0.0         UG    0      0        0 br0
10.200.0.0      0.0.0.0         255.255.255.0   U     0      0        0 br0
10.200.0.0      0.0.0.0         255.255.255.0   U     0      0        0 bridge0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 ens1f0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 ens1f1
169.254.0.0     0.0.0.0         255.255.0.0     U     1013   0        0 br0
169.254.0.0     0.0.0.0         255.255.0.0     U     1017   0        0 bridge0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 ens1f1
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
192.168.200.0   0.0.0.0         255.255.255.0   U     0      0        0 ens1f0
# route del -net 10.200.0.0 netmask 255.255.255.0 dev br0
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.200.0.10     0.0.0.0         UG    0      0        0 br0
10.200.0.0      0.0.0.0         255.255.255.0   U     0      0        0 bridge0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 ens1f0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 ens1f1
169.254.0.0     0.0.0.0         255.255.0.0     U     1013   0        0 br0
169.254.0.0     0.0.0.0         255.255.0.0     U     1017   0        0 bridge0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 ens1f1
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
192.168.200.0   0.0.0.0         255.255.255.0   U     0      0        0 ens1f0
```

edit KVM to connect with bridge0, and then check bridge if vnet exist from vm

```sh
# virsh shutdown lb-db-3
# virsh edit lb-db-3
<interface type='bridge'>
  <mac address='52:54:00:87:1d:74'/>
  <source bridge='bridge0'/>
  <model type='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
# virst start lb-db-3
# brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.fe5400abe5ef	no		vnet0
bridge0		8000.901b0e81e9e0	no		bond0
							vnet1
virbr0		8000.525400ecc5ec	yes		virbr0-nic

```


### Reference

[Instalation]http://unix-linux-server.blogspot.co.id/2014/10/centos-7-kvm-installation-and-bridge.html
