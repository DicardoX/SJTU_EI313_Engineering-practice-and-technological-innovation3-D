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

--------------

## 编译

- 进入qemu文件夹，新建一个build文件夹并进入；

- 设置编译选项，完成后发现`SDL support`缺失；

  ```
    ../configure --target-list=x86_64-softmmu --enable-kvm --enable-debug
  ```

```
  
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
 
