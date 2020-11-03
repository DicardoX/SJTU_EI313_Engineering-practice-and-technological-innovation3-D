# 课程大作业4：虚拟化性能对比测试及修改源码以降低损耗

---------------------

## 虚拟化性能对比测试

&emsp; 参见大作业2的测试结果即可。

---------------------

## `QEMU`源码分析：

[参考链接：QEMU 3.1.0 源码学习](https://abelsu7.top/2019/06/04/qemu-src-notes/)

&emsp; 在vl.c中定义了启动入口main函数（`void qemu_init(int argc, char **argv, char **envp)`），负责根据传入的参数例如RAM、CPU、devices来建立虚拟机的运行环境。CPU 的执行也是从此处开始的。
