# 利用`sysbench`进行性能测试

[参考链接：sysbench的安装和使用](https://www.cnblogs.com/chenshengkai/p/12626756.html)

-----------------

## Sysbench简介

&emsp; SysBench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。它主要包括以下几种方式的测试：
- cpu性能
- 磁盘io性能
- 调度程序性能
- 内存分配及传输速度 
- POSIX线程性能
- 数据库性能(OLTP基准测试)      

&emsp; 目前sysbench主要支持 MySQL,pgsql,oracle 这3种数据库。

----------------

## Ubuntu环境下Sysbench的安装

&emsp; 本次使用的虚拟环境为Ubuntu18.04。

### Sysbench的安装依赖

```
  sudo apt-get -y install make automake libtool pkg-config libaio-dev vim-common
```

### Sysbench的安装

```
  sudo apt-get install sysbench
```

### 验证sysbench安装是否成功

```
  dicardo@ubuntu:~$ sysbench --version
  sysbench 1.0.11
```

---------------

## 利用sysbench进行性能测试

### CPU测试：

```
  sysbench --test=cpu --cpu-max-prime=2000 run
```

### 线程测试：

```
  sysbench  --test=threads --num-threads=500 --thread-yields=100 --thread-locks=4 run
```

### IO测试：

#### （1）准备阶段：生成需要的测试文件，完成后会在当前目录下生成很多小文件。

```
  sysbench --test=fileio --num-threads=16 --file-total-size=2G --file-test-mode=rndrw prepare
```

#### （2）运行阶段：

```
  sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw run
```

#### （3）清理测试时生成的文件：

```
  sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw cleanup
```

### 内存测试：

```
  sysbench --test=memory --memory-block-size=8k --memory-total-size=1G run
```

### Mutex测试：

```
  sysbench –test=mutex –num-threads=100 –mutex-num=1000 –mutex-locks=100000 –mutex-loops=10000 run
```

### OLTP测试（数据库性能测试）：

&emsp; OLTP即联机事务处理过程，是对用户操作快速响应的方式之一。https://baike.baidu.com/item/OLTP/5019563?fr=aladdin

#### （1）准备阶段：生成需要的测试表

```
  sysbench --test=oltp --mysql-table-engine=innodb --mysql-host=10.0.0.8 --mysql-db=testsysbench --oltp-table-size=500000 --mysql-user=root --mysql password=Lad123456 prepare
```

#### （2）运行阶段：

```
  sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 run
```

#### （3）清理测试时生成的测试表：

```
  sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 cleanup
```


--------------













