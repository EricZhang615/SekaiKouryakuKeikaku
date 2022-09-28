# OpenStack初上手

## 从最小系统开始
[官方文档](https://docs.openstack.org/install-guide/index.html)
> ...create a minimum proof-of-concept for the purpose of learning about OpenStack.

上述教程包括如何在单机上使用手动安装（Manually）在使用多个VM作为node的情况下实现一个最小系统的搭建，包括Control node、Compute node等。

![](https://cdn.jsdelivr.net/gh/EricZhang615/myPics@main/img/202209282211476.png)

## 自动化部署工具：Kolla Ansible
[Kolla Ansible’s documentation](https://docs.openstack.org/kolla-ansible/yoga/)

## NFV服务：Tacker
[Tacker Documentation](https://docs.openstack.org/tacker/yoga/index.html)

* Tacker负责NFVO、VNFM、SFC构建的工作，VIM由OpenStack提供；
* **Tacker只能工作在`all-in-one`模式下，不能工作在`multinode`模式下**，所以多节点模式的平台不能使用Tacker（Multisite VIM Usage？）

目前的理解：一套OpenStack是运行在一个datacenter或者一个物理节点上的，如果要控制物理节点之间的转发，需要组OpenStack sites；Tacker上述的限制是指一个物理节点上现在还只支持`all-in-one`，但是是可以实现控制VNF在不同物理节点上的部署的？