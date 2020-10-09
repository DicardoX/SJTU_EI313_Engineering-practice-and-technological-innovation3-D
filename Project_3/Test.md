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
