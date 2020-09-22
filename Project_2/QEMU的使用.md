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
  dicardo@ubuntu:~/Desktop/qemu/build/x86_64-softmmu$ qemu-img create -f qcow2 ubuntu.img 20G
```

&emsp; 其中，`-f`用于指定镜像格式，`qcow2`是QEMU最常用的镜像格式，采用写时复制技术来优化性能。`ubunyu.img`是镜像的名字，`20G`是镜像文件大小（可使用`qemu-img resize ubuntu.img xxG`来重新设置镜像大小）。输出如下：

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

&emsp; QEMU使用KVM来提升虚拟机性能，如果不启用则会导致虚拟机性能损失。使用如下命令来进行检测，若有输出则代表硬件有虚拟化支持（否则关闭虚拟机， 在VMware中的虚拟机 -> 处理器和内存 -> 高级选项中勾选开启虚拟化）。

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

用如下指令启动带有ubuntu的虚拟机：

```
  sudo ./qemu-system-x86_64 -m 2048 -enable-kvm ubuntu.img -cdrom ./ubuntu-20.04.1-desktop-amd64.iso
```

&emsp; `-m`指定分配的虚拟机内存大小（默认单位是MB），`--enable-kvm`使用KVM进行加速，`-cdrom`添加ubuntu的安装镜像。可以在弹出的窗口中操作虚拟机，安装操作系统，安装后重启虚拟机便会从硬盘（ubuntu.img）启动。

&emsp; 之后要再启动虚拟机，只需要执行如下指令：

```
  sudo ./qemu-system-x86_64 -m 2048 -enable-kvm ubuntu.img
```

------------


## 补充：`.img`与`.iso`文件的关系：都是镜像文件后缀名，部分虚拟光驱不认`.iso`只认`.img`，因此要将`.iso`转换成`.img`使用

&emsp; 由于.ISO只能压缩使用ISO9660和UDF这两种文件系统的存储媒介，意即.ISO只能拿来压缩CD或DVD，因此才发展出了.IMG，它是以.ISO格式为基础另外新增可压缩使用其它文件系统的存储媒介的能力，.IMG可向后兼容于.ISO，如果是拿来压缩CD或DVD，则使用.IMG和.ISO这两种格式所压缩出来的内容是一样的。img格式的打开方式可以是光盘刻录，也可以用软件解压。IMG可以做为以下用途：数字存储、传输、以及整片软盘内容的复制，可挂载到虚拟软盘上。

&emsp; 都是光盘的镜像文件，ISO可以用虚拟光驱软件来打开。IMG可以用HDCOPY或者DAEMON软件。其使用效果都是一样的。但是不同的虚拟光驱它所使用的镜象包扩展名也不一样的，所以你要压缩光盘的时候，必须要针对你所使用的虚拟光驱软件来做。ISO是比较通用的光盘镜象模式，但是往往有一些虚拟光驱不认这种格式。所以在不认img镜像文件的时候，可以将img后缀名修改成iso来打开使用。







