# KVM / virsh
Setup and connect


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
# virsh shutdown vm
# virsh dumpxml vm > /tmp/vm.xml
# scp /tmp/vm.xml kvm02:/tmp/vm.xml
# scp /var/lib/libvirt/images/vm.qcow2 kvm02:/var/lib/libvirt/images/vm.qcow2
```
on destination host

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

### Reference

[Instalation]https://www.paulmellors.net/virsh-console-centos7/
