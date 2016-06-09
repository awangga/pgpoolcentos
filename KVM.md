# KVM / virsh
Setup and connect

### Start Reboot Shutdown, plug power

```sh
# virsh start vm
# virsh reboot vm
# virst shutdown vm
# virsh destroy vm
```


### Connect to shell via console

on your vm os
```sh
# grubby --update-kernel=ALL --args="console=ttyS0"
# reboot
```

connect from your virsh host with console, press enter once

```sh
# virsh console lb-db-4
Connected to domain lb-db-4
Escape character is ^]

CentOS Linux 7 (Core)
Kernel 3.10.0-229.20.1.el7.x86_64 on an x86_64

lb-db-4 login:
```

### KVM Migration
On source host

```sh
# virsh list --all
# virsh shutdown vm
# virsh dumpxml vm > /tmp/vm.xml
# scp /tmp/vm.xml kvm02:/tmp/vm.xml
# scp /var/lib/libvirt/images/vm.qcow2 kvm02:/var/lib/libvirt/images/vm.qcow2
```
on destination host

edit xml first change UUID, mac address, name, and disk
#### Generate UUID, MAC address

```sh
# echo $(uuidgen)
# echo $(echo $FQDN|md5sum|sed 's/^\(..\)\(..\)\(..\)\(..\)\(..\).*$/02:\1:\2:\3:\4:\5/')
```

```sh
# virsh define /tmp/vm.xml 
# virsh start vm
```
remove delete old vm on source host

```sh
# virsh destroy vm
# virsh undefine vm
# rm /var/lib/libvirt/images/vm.qcow2
```

### Attach virtual disk image for data (vdb)
using --persistent option for updating xml

```sh
# virsh attach-disk postgres-04 --source /pgdata/postgres-04-data.img --target vdb --persistent
```

### change ip address and hostname
change ip address

```sh
$ systemctl status NetworkManager.service
$ nmcli dev status
$ vi /etc/sysconfig/network-script/ifcfg-eth0
ONBOOT="yes"
BOOTPROTO="static"
IPADDR=192.168.0.17
NETMASK=255.255.255.0
NM_CONTROLLED=no
```
change hostname

```sh
# hostnamectl status
# hostnamectl set-hostname lb-db3 --static
# hostnamectl set-hostname lb-db3 --static
# hostnamectl set-hostname lb-db-3 --transient
# reboot
```

### VM routing filter

```sh
# virsh nwfilter-list
```

### Removing network filter 
if you can't reach other host in other KVM just remove all network filter

```sh
# virsh  nwfilter-list
 UUID                                  Name                 
------------------------------------------------------------------
 611a47b3-9934-445d-8adc-796fca6f46fb  no-mac-broadcast    
 913ecfa4-f48d-485b-b578-b0eaef6f4fc4  no-mac-spoofing     
 d4325af6-9e8c-4449-a073-b3aa20554fe4  qemu-announce-self  
 0eb94d1e-25df-43b8-b6bf-ad94143183d6  qemu-announce-self-rarp

# virsh nwfilter-undefine no-mac-broadcast
Network filter no-mac-broadcast undefined

# virsh nwfilter-undefine no-mac-spoofing
```
add arp broadcast or you can set arp static by command

```sh
# arp -i eth0 -s 10.200.0.202 02:68:b3:29:da:98
# arp -i eth0 -s 10.200.0.203 52:54:00:87:1d:75
# arp -i eth0 -s 10.200.0.220 02:6f:ae:23:0a:7b
# arp -i eth0 -s 10.200.0.221 02:44:3e:c3:61:0c
# arp -i eth0 -s 10.200.0.222 52:54:00:53:5b:c2
# arp -i eth0 -s 10.200.0.223 52:54:00:5f:b2:c5
```
or you can create shell script for running every minute on crontab 
* * * * * /root/arp.sh

```sh
#!/bin/sh

arp -i eth0 -s 10.200.0.222 52:54:00:53:5b:c2
arp -i eth0 -s 10.200.0.223 52:54:00:5f:b2:c5
arp -i eth0 -s 10.200.0.220 02:6f:ae:23:0a:7b
arp -i eth0 -s 10.200.0.221 02:44:3e:c3:61:0c
arp -i eth0 -s 10.200.0.202 02:68:b3:29:da:98 
arp -i eth0 -s 10.200.0.203 52:54:00:87:1d:75
```

### Reference

[Instalation]https://www.paulmellors.net/virsh-console-centos7/
