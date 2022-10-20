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

# Kolla-Ansible部署时libvirt container报错

## 问题描述

执行
```
(venv) # kolla-ansible -i all-in-one deploy -vvvv
```
报错
```
fatal: [localhost]: FAILED! => {"censored": "the output has been hidden due to the fact that 'no_log: true' was specified for this result", "changed": true}
```

## 解决方法

不要用-vvvv 用-vvv 然而并没有解决问题 这次直接在这一步卡住了 见下一个问题

# Kolla-Ansible部署时卡在`Create libvirt SASL user`这一步

## 问题描述

执行
```
(venv) # kolla-ansible -i all-in-one deploy -vvv
```
一直卡在`Create libvirt SASL user`这一步，于是去看log文件`/var/log/kolla/libvirt/libvirt.log`，发现报错：
```
2022-10-20 08:00:03.248+0000: 7647: info : libvirt version: 6.0.0, package: 0ubuntu8.16 (Marc Deslauriers <marc.deslauriers@ubuntu.com> Wed, 20 Apr 2022 11:31:12 -0400)
2022-10-20 08:00:03.248+0000: 7647: info : hostname: ubuntu
2022-10-20 08:00:03.248+0000: 7647: error : virPidFileAcquirePath:367 : Failed to acquire pid file '/run/libvirtd.pid': Resource temporarily unavailable
2022-10-20 08:01:03.544+0000: 8558: info : libvirt version: 6.0.0, package: 0ubuntu8.16 (Marc Deslauriers <marc.deslauriers@ubuntu.com> Wed, 20 Apr 2022 11:31:12 -0400)
2022-10-20 08:01:03.544+0000: 8558: info : hostname: ubuntu
2022-10-20 08:01:03.544+0000: 8558: error : virPidFileAcquirePath:367 : Failed to acquire pid file '/run/libvirtd.pid': Resource temporarily unavailable
2022-10-20 08:02:03.820+0000: 9373: info : libvirt version: 6.0.0, package: 0ubuntu8.16 (Marc Deslauriers <marc.deslauriers@ubuntu.com> Wed, 20 Apr 2022 11:31:12 -0400)
2022-10-20 08:02:03.820+0000: 9373: info : hostname: ubuntu
2022-10-20 08:02:03.820+0000: 9373: error : virPidFileAcquirePath:367 : Failed to acquire pid file '/run/libvirtd.pid': Resource temporarily unavailable
```
同时查看宿主机libvirt服务的日志也有同样的报错：
```
# libvirt -v
2022-10-20 08:04:24.949+0000: 11582: info : libvirt version: 4.0.0, package: 1ubuntu8.21 (Marc Deslauriers <marc.deslauriers@ubuntu.com> Wed, 20 Apr 2022 13:18:06 -0400)
2022-10-20 08:04:24.949+0000: 11582: info : hostname: ubuntu
2022-10-20 08:04:24.949+0000: 11582: info : virObjectNew:254 : OBJECT_NEW: obj=0x5574129a2850 classname=virAccessManagerClass
2022-10-20 08:04:24.949+0000: 11582: info : virObjectNew:254 : OBJECT_NEW: obj=0x5574129918b0 classname=virAccessManagerClass
2022-10-20 08:04:24.949+0000: 11582: info : virObjectRef:388 : OBJECT_REF: obj=0x5574129a2850
2022-10-20 08:04:24.949+0000: 11582: info : virObjectUnref:350 : OBJECT_UNREF: obj=0x5574129a2850
2022-10-20 08:04:24.949+0000: 11582: error : virPidFileAcquirePath:422 : Failed to acquire pid file '/var/run/libvirtd.pid': Resource temporarily unavailable
2022-10-20 08:04:24.949+0000: 11582: info : virNetlinkEventServiceStopAll:781 : stopping all netlink event services
2022-10-20 08:04:24.949+0000: 11582: info : virNetlinkEventServiceStop:744 : stopping netlink event service
```

## 解决方法

https://blog.csdn.net/wuliangtianzu/article/details/109489649

这个问题是因为宿主机上起了libvirtd服务，和容器服务相冲突，所以要关掉宿主机的libvirt服务，然后重新部署。
```
# service libvirtd stop
```
重新部署前清除已经启动的服务和docker
```
kolla-ansible destroy -i all-in-one --yes-i-really-really-mean-it
```
其他可能有用的工具：

* kolla-ansible prechecks：在执行部署命令之前，先检查环境是否正确；

* tools/cleanup-containers：可用于从系统中移除部署的容器；

* tools/cleanup-host：可用于移除由于网络变化引发的Docker启动的neutron-agents主机；

* tools/cleanup-images：可用于从本地缓存中移除所有的docker image。