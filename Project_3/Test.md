# Test

--------------

[安装过程](https://blog.csdn.net/xiaohui5319/article/details/11284111)

[libnl-dev依赖包补充](https://blog.csdn.net/ever_peng/article/details/80161863)

`No package 'libnl-route-3.0' found` [解决方案](https://stackoverflow.com/questions/16737351/jhbuild-install-networkmanager-ubuntu-13-04)

`configure: error: You must install the libyajl library & headers to compile libvirt`[解决方案](https://blog.51cto.com/itech/1812394?utm_source=oschina-app)

`./configure: 41: ./configure: cmake: not found`

需要安装cmake:

```
  sudo apt-get install cmake
  sudo apt-get install cmake-qt-gui
```
&emsp; 之后分别执行：

```
  ./configure
  make
  sudo make install
```
（makefile发生错误后）更改为`libvirt-4.4.0`版本.

`./configure`遇到`configure: error: "xsltproc is required to build libvirt"`错误。解决方案`sudo apt-get install xsltproc`

完成`./configure`后开始`make`,完成。

之后开始`sudo make install`,完成。

`reboot`重启。

下载`Ubuntu18.04`镜像，其间顺便安装一下`sudo apt-get install virt-manager`。一直报错，决定从一开始就先安装好`virt-manager`.

`./configure`遇到`configure: error: libnl-devel >= 1.1 is required for macvtap support`错误。解决方案参见[libnl-dev依赖包补充](https://blog.csdn.net/ever_peng/article/details/80161863)

`No package 'libnl-route-3.0' found`解决方案见上。

`configure: error: libxml2 >= 2.6.0 is required for libvirt`错误。[解决方案](http://blog.sina.com.cn/s/blog_896b31c701013qc2.html)（一共两个指令）。

`configure: error: You must install the pciaccess module to build with udev`错误。[解决方案](https://blog.csdn.net/heybob/article/details/24481397)

`configure: error: You must install device-mapper-devel/libdevmapper to compile libvirt with mpath storage driver`

解决方法：`sudo apt-get install libdevmapper-dev`[解决方案](http://blog.sina.com.cn/s/blog_896b31c701013qc2.html)

`./configure`完成，开始`make`


(放弃从源码编译安装)

```
  sudo apt-get install virt-manager
  sudo apt-get install virtinst
  sudo apt-get install libvirt-bin
  sudo apt-get install libvirt-dev
```







