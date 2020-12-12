# 工程实践与科技创新大作业5

--------------

## `virsh`配置虚拟机

[参考链接：Linux实现KVM+QEMU+libvirt的虚拟机环境 并使用virsh对虚拟机进行管理](https://blog.csdn.net/qq_24369113/article/details/52929439)

&emsp; 在直接使用`virt-manager`创建虚拟机时报错，因此决定采用`qemu-img create` + `virsh define ubuntu.xml`的方式进行配置（所使用的xml配置文件见`/etc/libvirt/qemu/`，必须要先`virsh define demo.xml`才能生成，分析见[KVM 虚拟机 XML 配置文件详解](https://blog.51cto.com/4746316/2336524)）。

&emsp; `virsh define demo.xml`之后，`virsh start ubuntu1`开启虚拟机，并使用`virt-manager`来管理虚拟机 + 进入虚拟机。

&emsp; 注意：当虚拟机中出现`graphics initialization failed Error setting up gfxboot`的报错时，只需要键入`help`，进入界面后按`enter`即可开始安装。
