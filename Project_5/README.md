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

### 3. 利用`usertools/dpdk-devbind.py`绑定网卡

 - 安装`net-tools`，使用`ifconfig`查看网卡配置，发现目前kvm虚拟机只有一个网卡，因此需要再源主机上再为其创建一个。源主机本身就有一个`virbr0`，按照[参考链接二](https://www.cnblogs.com/wangyong-blog/p/11148133.html)的方式为kvm虚拟机添加一个新的网卡。
 
 - `ifconfig [name] down`将网卡解绑（name为网卡名称）
 
 - 加载`uio_pci_generic`功能：`sudo modprobe uio_pci_generic`
 
 - 绑定网卡到`uio_pci_generic`下：`./dpdk-devbind.py --bind=uio_pci_generic [name]`
 
 - 查看kvm虚拟机网卡设备状态：`./dpdk-devbind.py --status`
 
---------------

### 4. 配置`HugePage`

&emsp; 进入`usertools`文件夹后`./dpdk-setup.sh`。一开始选择`pages`数目为64，在运行helloworld时报错无法获得page资源，因此修改为512，解决。

--------------

### 5. `examples/helloworld`的编译和运行

&emsp; 进入`examples/helloworld`，进行如下配置：

```
export RTE_SDK=/home/lianpeng/dpdk-19.11（此处为DPDK的解压目录，改成自己的）
export RTE_TARGET=x86_64-native-linuxapp-gcc（改成自己的DPDK环境配置）
```

&emsp; 然后编译`make`，会有如下输出：

```
CC main.o
LD helloworld
INSTALL-APP helloworld
INSTALL-MAP helloworld.map
```

&emsp; 进入`build`目录，运行`sudo ./helloworld`（注意，要在root权限下运行，否则会报错），会有输出（最后一句为`hello from core 0`，取决于kvm虚拟机有多少个vcpu）。

&emsp; 在最后运行时可能会报`cpu type`不支持`SSSE3`或者`SSSE4_1`指令集，此时关闭虚拟机，在`virt-manager`中将cpu类型改为`Opteron_G5`即可（好多可用的虚拟cpu型号并不显示在下拉菜单里，可以在源主机中用`qemu-system-x86_64 -cpu help`指令来查看可用的cpu型号）。

&emsp; 至此，kvm虚拟机上的`dpdk`库安装完成。
 
 
&emsp; 相关依赖包：

```
sudo apt install linux-headers-$(uname -r) 
sudo apt install build-essential git gcc 
pip3 install meson ninja
sudo apt install python3-pip meson
sudo apt install libpcap-dev liblz4-dev liblz4-tool
sudo apt install libnuma-dev libarchive-dev
sudo apt install libisal-dev libfdt-dev libbpfcc-dev 
sudo apt install libssl-dev libcrypto++6 libelf-dev  
sudo apt install wireshark
```
 
```
meason build
cd build
ninja
```
 
```
ninja install
sudo ldconfig
export PKG_CONFIG_PATH=/usr/local/lib/x86_64-linux-gnu/pkgconfig
```
 
```
qemu-system-x86_64 -cpu help
```
 
```
sudo su
echo 512 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages 
mkdir /mnt/huge
mount -t hugetlbfs nodev /mnt/huge
```

```
cat /proc/meminfo | grep Huge
```

```
sudo modprobe uio
sudo insmod igb_kio.ko
sudo modprobe uio_pci_generic 
sudo modprobe vfio-pci
```

```
sudo ./usertools/dpdk-devbind.py --force -b igb_uio 0000:00:03.0
```
 
```
meson build
cd build
ninja
ninja install
```
 
```
description = 'A Pktgen custom configuration'
# Setup information
setup = {
        'exec': (
                'sudo', '-E'
        ),
        'devices': (
                '00:03.0',                                       # 网卡设备
        ),
        # UIO module type, igb_uio, vfio-pci or uio_pci_generic
        'uio': 'igb_uio'                                         # uio支持的内核模块
}

# Run options
run = {
      'exec': ('sudo', '-E'),
      'app_name': 'pktgen',                                      # 名称
      'app_path': (
              './usr/local/bin/%(app_name)s',                    # 路径
              '/usr/local/bin/%(app_name)s'
      ),
      'cores': '0-1',                                            # 使用两个逻辑内核
      'nrank': '5',                                              # 开启5个内存通道
      'proc': 'auto',
      'log': '7',
      'prefix': 'pg',
      'blocklist': (),                                           # 黑名单设为空
      'allowlist': (
              '00:03.0',                                         # 允许的网卡设备
      ),
      'opts': (
              '-v',
              '-T',
              '-P',                                              # 开启网卡混杂模式
              '-j',
      ),
      'map': (
              '1.0',
      ),
      'theme': 'themes/black-yellow.theme'                       # 主题
 }
```

```
sudo ./tool/run.py -s mytest
sudo ./tool/run.py mytest
```

```
run = {
        ...,
        'plugin': '/usr/local/lib/x86_64-linux-gnu/',
        ...
}
```



 
