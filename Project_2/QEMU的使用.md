# QEMU的使用

[参考链接：使用QEMU创建虚拟机](http://www.360doc.com/content/17/0531/22/6080914_658855867.shtml)

--------

## 进入相应文件夹

&emsp; 进入`build`文件夹下的`x86_64-softmmu`文件夹，查看文件目录：

```
  dicardo@ubuntu:~/Desktop/qemu/build/x86_64-softmmu$ ls
  config-target.mak  description-pak  qemu-system-x86_64
```
&emsp; 启动qemu虚拟机的方式是在该目录下输入如下指令：

```
  ./qemu-system-x86_64          // 会启动默认镜像下的操作系统，否则在后面加上特定的镜像名称.img
```

---------

## 使用`qemu-img`指令创建虚拟机镜像

```
  dicardo@ubuntu:~/Desktop/qemu/build/x86_64-softmmu$ qemu-img create -f qcow2 ubuntu.img 10G
```

&emsp; 其中，`-f`用于指定镜像格式，`qcow2`是QEMU最常用的镜像格式，采用写时复制技术来优化性能。`ubunyu.img`是镜像的名字，`10G`是镜像文件大小。输出如下：

```
  Formatting 'ubuntu.img', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=10737418240 lazy_refcounts=off refcount_bits=16
```

&emsp; 镜像文件创建完成后，可使用如下指令来启动x86架构的虚拟机:

```
  ./qemu-system-x86_64 ubuntu.img
```

&emsp; 此时会弹出窗口作为虚拟机的显示器。因为ubuntu.img并未给虚拟机安装操作系统，故会显示"No bootable device"。

-----------

## 准备操作系统镜像

&emsp; 从Ubuntu官网获取Ubuntu20.04版本的下载地址，并使用如下指令进行下载：

```
  dicardo@ubuntu:~/Desktop/qemu/build/x86_64-softmmu$ wget https://releases.ubuntu.com/20.04.1/ubuntu-20.04.1-desktop-amd64.iso
  --2020-09-22 19:19:59--  https://releases.ubuntu.com/20.04.1/ubuntu-20.04.1-desktop-amd64.iso
  正在解析主机 releases.ubuntu.com (releases.ubuntu.com)... 91.189.88.247, 91.189.88.248, 91.189.91.124, ...
  正在连接 releases.ubuntu.com (releases.ubuntu.com)|91.189.88.247|:443... 已连接。
  已发出 HTTP 请求，正在等待回应... 200 OK
  长度： 2785017856 (2.6G) [application/x-iso9660-image]
  正在保存至: “ubuntu-20.04.1-desktop-amd64.iso”
  desktop-amd64.iso     6%[>                   ] 185.21M  1.16MB/s    剩余 29m 52s
```

&emsp; 若过程中出现“段错误，核心已转储”，则使用如下命令从中断处继续下载：

```
  wget -c  https://releases.ubuntu.com/20.04.1/ubuntu-20.04.1-desktop-amd64.iso
```

&emsp; 直到出现“文件下载已完成；不会进行任何操作“的提示。


------------

## 检查KVM是否可用

&emsp; QEMU使用KVM来提升虚拟机性能，如果不启用则会导致虚拟机性能损失。使用如下命令来进行检测，若有输出则代表硬件有虚拟化支持（否则关闭虚拟机， 在VMware中的虚拟机 -> 处理器和内存 -> 高级选项中勾选开启虚拟化）.

```
  grep -E 'vmx|svm' /proc/cpuinfo
```
&emsp; 其次是查看KVM模块是否已经加载：

```
  lsmod | grep kvm
```
&emsp; 若显示：

```
  kvm_intel             253952  0
  kvm                   655360  1 kvm_intel
```
&emsp; 则说明kvm模块已经加载。最后只需要保证在编译执行configure脚本时enable了kvm即可。

-----------

## 启动虚拟机安装操作系统












