# 工程实践与科技创新大作业5

--------------

## `virsh`配置虚拟机

[参考链接：Linux实现KVM+QEMU+libvirt的虚拟机环境 并使用virsh对虚拟机进行管理](https://blog.csdn.net/qq_24369113/article/details/52929439)

&emsp; 在直接使用`virt-manager`创建虚拟机时报错，因此决定采用`qemu-img create` + `virsh define ubuntu.xml`的方式进行配置（所使用的xml配置文件见`/etc/libvirt/qemu/`，必须要先`virsh define demo.xml`才能生成，分析见[KVM 虚拟机 XML 配置文件详解](https://blog.51cto.com/4746316/2336524)）。

&emsp; `virsh define demo.xml`之后，`virsh start ubuntu1`开启虚拟机，并使用`virt-manager`来管理虚拟机 + 进入虚拟机。

&emsp; 注意：当虚拟机中出现`graphics initialization failed Error setting up gfxboot`的报错时，只需要键入`help`，进入界面后按`enter`即可开始安装。注意！这样的情况是由于每次都是从光盘启动，因此每次重启都会重新安装UbuntuOS，应该在虚拟机设置中改为从磁盘启动！

## `dpdk`安装

[参考链接一（主要）：ubuntu-18.04的DPDK-19.11安装过程](https://blog.csdn.net/qq_39992615/article/details/103777991?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)


[参考链接二：kvm虚拟机添加网卡](https://www.cnblogs.com/wangyong-blog/p/11148133.html)

[参考链接三：]

### 1. 依赖包安装

&emsp; 进入解压后的`dpdk-19.08`内：

 - `sudo apt-get install libpcap-dev`
 
 - Enable pcap: `make config T=x86_64-native-linuxapp-gcc` + `sed -ri 's,(PMD_PCAP=).*,\1y,' build/.config`
 
 - `sudo apt-get install libnuma-dev`
 
 - 编译：`make`
 
---------------

### 2. 利用`usertools/dpdk-setup.sh`配置环境

&emsp; 进入`usertools`文件夹后`./dpdk-setup.sh`。

 - 第一步先选择虚拟机对应的dpdk环境，这是选择`x86_64-native-linuxapp-gcc`
 
 - 设置Linux环境：这里选择加载`IGB UIO`和`VFIO`模块

---------------

## 3. 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
