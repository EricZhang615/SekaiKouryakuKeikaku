# 如何在Windows系统上将SSH公钥设置在远端Linux系统上

目前，Windows 10 的 OpenSSH 客户端实现没有ssh-copy-id可用的命令。但是，PowerShell 单行命令可以模拟该ssh-copy-id命令，并允许您将 ssh-keygen 命令生成的 SSH 公钥复制到远程 Linux 设备以进行无密码登录。

## 将 SSH 密钥复制到远程 Linux 设备

使用下面的PowerShell单行命令将`id_rsa.pub`公钥的内容复制到远程Linux设备。

```(bash)
type id_rsa.pub | ssh root@123.123.123.123 -p 12345 "cat >> .ssh/authorized_keys"
```