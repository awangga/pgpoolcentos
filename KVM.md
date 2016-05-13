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
# virsh undefine vm
# rm /var/lib/libvirt/images/vm.qcow2
```

### Reference

[Instalation]https://www.paulmellors.net/virsh-console-centos7/
