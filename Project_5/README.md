# 工程实践与科技创新大作业5

--------------

## `virsh`配置虚拟机

[参考链接：Linux实现KVM+QEMU+libvirt的虚拟机环境 并使用virsh对虚拟机进行管理](https://blog.csdn.net/qq_24369113/article/details/52929439)

&emsp; 所使用的xml配置文件见`demo.xml`。

&emsp; `virsh define demo.xml`之后，`virsh start ubuntu1`开启虚拟机，并使用`virt-manager`来管理虚拟机 + 进入虚拟机。
