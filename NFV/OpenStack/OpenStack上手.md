# OpenStack初上手

## 从最小系统开始
[官方文档](https://docs.openstack.org/install-guide/index.html)
> ...create a minimum proof-of-concept for the purpose of learning about OpenStack.

上述教程包括如何在单机上使用手动安装（Manually）在使用多个VM作为node的情况下实现一个最小系统的搭建，包括Control node、Compute node等。

![](https://cdn.jsdelivr.net/gh/EricZhang615/myPics@main/img/202209282211476.png)

## 自动化部署工具：Kolla Ansible
[Kolla Ansible’s documentation](https://docs.openstack.org/kolla-ansible/yoga/)

[Install Tacker via Kolla Anisible](https://docs.openstack.org/tacker/yoga/install/kolla.html)

## NFV服务：Tacker
[Tacker Documentation](https://docs.openstack.org/tacker/yoga/index.html)

[Tacker部署与使用详解](https://blog.csdn.net/tpiperatgod/article/details/56282617)

* Tacker负责NFVO、VNFM、SFC构建的工作，VIM由OpenStack提供；
* **Tacker只能工作在`all-in-one`模式下，不能工作在`multinode`模式下**，所以多节点模式的平台不能使用Tacker（[Multisite VIM Usage](https://docs.openstack.org/tacker/yoga/user/multisite_vim_usage_guide.html)？[K8s VIM Installation](https://docs.openstack.org/tacker/yoga/install/kubernetes_vim_installation.html)）

目前的理解：一套OpenStack是运行在一个datacenter或者一个物理节点上的，如果要控制物理节点之间的转发，需要组OpenStack sites；Tacker上述的限制是指一个物理节点上现在还只支持`all-in-one`，但是是可以实现控制VNF在不同物理节点上的部署的？

## OpenStack NFV架构的疑惑
对于一个NFV网络，物理节点与物理节点之间应该是异地的、分离的，那么在OpenStack中，这些节点应该作为一个OpenStack `multinode`系统中的多个node吗？还是每一个物理节点单独为一个OpenStack `all-in-one`系统吗？

![](https://cdn.jsdelivr.net/gh/EricZhang615/myPics@main/img/202210041418828.png)

如果属于前者，由统一的OpenStack系统生成并管理一个VIM，然后由Tacker在VIM基础上进行VNF的管理和编排；然而Tacker不支持在`multinode`下工作。如果采用后者，那么属于OpenStack multisite技术，每一个节点提供一个VIM，管理难度要比前者大，节点和节点之间完全分离。通过Multisite VIM，Tacker可以在没有安装Tacker服务的OpenStack站点上部署VNF。

[OpenStack NFV架构简述](https://www.slideshare.net/TrinathSomanchi/demystifying-openstack-for-nfv)

在我看来，此时的关键点在于ESTI NFV架构中，不同的物理节点是否是各自独立的VIM？对于卫星组网环境下，那么应该是属于第二类情况。

## Multiple Regions部署
所谓openstack多region，就是多套openstack共享一个keystone和horizon。每个区域一套openstack环境，可以分布在不同的地理位置，只要网络可达就行。每个region都有个完整的Openstack部署环境，有自己的一套服务的endpoint（服务入口）。不同的region共享一套keystone和horizon来提供访问控制与web操作，regions之间完全隔离，但是多个regions之间共享同一个keystone和dashboard。

[Multiple Regions Deployment with Kolla](https://docs.openstack.org/kolla-ansible/yoga/user/multi-regions.html)

![](https://cdn.jsdelivr.net/gh/EricZhang615/myPics@main/img/202209291731479.png)


TO DO:

* 在两台server分别搭两个`all-in-one`的OpenStack服务，其中一台做主机组多region，每台server自己处于一个region中
* VNF是如何定义的 如何编写自定义的VNF 如何在tacker服务上部署VNF
* SFC是如何定义的 如何在tacker服务上部署SFC？相关内容：VNF Forward Graph(VNFFG), VNF Forward Grapf descriptor(VNFFGD)... [OpenStack tacker and service function chaining sfc with kolla
](https://blog.egonzalez.org/openstack/index/openstack-tacker-and-service-function-chaining-sfc-with-kolla)