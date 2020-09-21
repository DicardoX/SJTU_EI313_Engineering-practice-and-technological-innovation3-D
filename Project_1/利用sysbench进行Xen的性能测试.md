# 利用`sysbench`进行Xen的性能测试

[参考链接：sysbench的安装和使用](https://www.cnblogs.com/chenshengkai/p/12626756.html)

-----------------

## Sysbench简介

SysBench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。它主要包括以下几种方式的测试：
       1、cpu性能
       2、磁盘io性能
       3、调度程序性能
       4、内存分配及传输速度 
       5、POSIX线程性能
       6、数据库性能(OLTP基准测试)      

目前sysbench主要支持 MySQL,pgsql,oracle 这3种数据库。

