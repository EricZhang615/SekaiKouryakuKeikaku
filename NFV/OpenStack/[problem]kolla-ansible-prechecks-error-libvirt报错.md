# Kolla-Ansible部署前执行`kolla-anisible prechecks`时nova在`Checking that libvirt is not running`报错

## 问题描述

执行
```
(venv) # kolla-ansible prechecks
```
在`TASK [nova-cell : Checking that libvirt is not running]`报错

## 解决方法

```
rm /var/run/libvirt/libvirt-sock -f
```
可能原因是libvirt要由kolla-ansible在部署时安装，不能提前安装，因此OpenStack部署最好是在全新的系统上并在最小安装系统上进行更好