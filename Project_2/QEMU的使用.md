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
  ./qemu-system-x86_64
```

---------

## 使用`qemu-img`指令创建虚拟机镜像



