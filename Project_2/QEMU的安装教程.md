# QEMU的安装教程

[QEMU安装教程链接](https://www.jianshu.com/p/413895d9b8cf)

------------

## 依赖包的安装

```
  ##安装依赖
  sudo apt-get install build-essential pkg-config zlib1g-dev
  sudo apt-get install libglib2.0-0 libglib2.0-dev
  sudo apt-get install libsdl1.2-dev
  sudo apt-get install libpixman-1-dev libfdt-dev
  sudo apt-get install autoconf automake libtool
  sudo apt-get install librbd-dev
  apt-get install libaio-dev
  apt-get install flex bison
```

--------------

## 下载QEMU源码并解压zip

```
  链接: https://pan.baidu.com/s/1ovUVVt0YX44MlZqrrYiy-w  密码: 7f32
```

&emsp; 或在命令行输入如下指令：

```
  wget https://download.qemu.org/qemu-5.1.0.tar.xz
  tar xvJf qemu-5.1.0.tar.xz
```

--------------

## 编译

- 进入qemu文件夹，新建一个build文件夹并进入；

- 设置编译选项，完成后发现`SDL support`缺失（在一大串输出的中间）；

  ```
    ../configure --target-list=x86_64-softmmu --enable-kvm --enable-debug
  ```

&emsp; 作用分别是：

```
  --enable-kvm：编译KVM模块，使QEMU可以使用KVM来访问硬件提供的虚拟化服务
  --enable-vnc：启用VNC（虚拟网络控制台的缩写）
  --enable-werror：编译时，将所有warning当作error处理
  --target-list：选择目标机器的架构。默认是将所有的架构都编译，但指定之后可以更快地编译
```


- 安装SDL2，[安装教程](https://www.jianshu.com/p/17ff0f40ec08)

  ```
    sudo apt-get install libsdl2-2.0

    sudo apt-get install libsdl2-dev

    sudo apt-get install libsdl2-mixer-dev

    sudo apt-get install libsdl2-image-dev

    sudo apt-get install libsdl2-ttf-dev

    sudo apt-get install libsdl2-gfx-dev

  ```
  -
- 重新设置编译选项，观察到SDL support显示为yes；

  ```
    ../configure --target-list=x86_64-softmmu --enable-kvm --enable-debug
  ```
  
- 编译；
  ```
    make
  ```
 
