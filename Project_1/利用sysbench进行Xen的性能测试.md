# 利用`sysbench`进行Xen的性能测试

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

# Ubuntu环境下Sysbench的安装

&emsp; 本次使用的虚拟环境为Ubuntu18.04。

