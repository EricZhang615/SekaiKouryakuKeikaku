# 一次系统资源占用过高的排查过程

## 问题描述

发现Antimalware Service Executable和Local security authority process的cpu占用过高，合计达到100%，同时发现sshd有不明网络活动，于是停止sshd服务，问题解决；但是只要sshd一启动，就会有大量的sshd进程频繁的启动和结束，并且以3mbps上传和下载，进而导致Antimalware Service Executable和Local security authority process占用上升。

通过`netstat -a`查看，发现在内网有多个ip与本机的22端口建立了连接（校园网10.112.0.0/16），并且每过一段时间，ip就会变化（不是同一个ip在建立连接）；sshd服务关闭后，连接的频繁建立也停止了。

查看启动的sshd进程，看不到是由其他程序启动的：

- 路径名称就是sshd的安装位置
- 启动命令行是`"..\sshd.exe" -R`和`"..\sshd.exe" -y`两种
- 启动用户有的显示的是SYSTEM，有的是`sshd_*****`这种格式，后面是数字
- 每一个sshd进程启动后迅速就结束了，没能定位到父进程pid

上面的情况是直接网线连到10.112.0.1网关上发生的，当切换到一个位于同网段的路由器后（路由器分配了192的内网地址），就不会出现这种现象了。