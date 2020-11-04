# 课程大作业4：虚拟化性能对比测试及修改源码以降低损耗

---------------------

## 虚拟化性能对比测试

&emsp; 参见大作业2的测试结果即可。

---------------------

## `QEMU`源码分析：

[参考链接一：QEMU 3.1.0 源码学习](https://abelsu7.top/2019/06/04/qemu-src-notes/)

[参考链接二：QEMU架构浅析](https://cloud.tencent.com/developer/article/1521505)

[参考链接三：Linux 下 kvm 虚拟化 Windows 的几个性能优化建议](https://v2ex.com/t/607276)

[参考链接四：QEMU入门指南](https://blog.csdn.net/FontThrone/article/details/104157859)

&emsp; 从`QEMU`的角度来说，`QEMU（Quick Emulator)`本身并不包含或依赖KVM模块，而是一套由`Fabrice Bellard`编写的模拟计算机的自由软件。`QEMU`虚拟机是一个纯软件的实现，可以在没有`KVM`模块的情况下独立运行，但是性能比较低。`QEMU`有整套的虚拟机实现，包括处理器虚拟化、内存虚拟化以及`I/O`设备的虚拟化。在不需要`KVM`加速的情况下，`QEMU`通过一个特殊的“重编译器”对特定的处理器的二进制代码进行翻译，从而具有了跨平台的通用性。`QEMU`有两种工作模式：系统模式，可以模拟出整个电脑系统，另一种是用户模式，可以运行不同与当前硬件平台的其他平台上的程序（比如在`x86`平台上运行跑在`ARM`平台上的程序）。目前最新版本是`4.x`。从`QEMU`角度来看，虚拟机运行期间，`QEMU`通过`KVM`模块提供的系统调用接口进行内核设置，由`KVM`模块负责将虚拟机置于处理器的`VMX`模式运行。`QEMU`使用了`KVM`模块的虚拟化功能，为自己的虚拟机提供硬件虚拟化加速以提高虚拟机的性能。

&emsp; `cpu-exec.c`和`cpus.c`文件依次上传了`qemu-3.0.1`和`qemu-5.1.0`两个版本的`cpu-exec.c` / `cpus.c`文件，以此使用`History`代码比对功能来观察不同版本文件的变化。

&emsp; 我们使用`qemu-3.0.1`版本作为母版进行修改，原因是更高版本的`qemu`代码优化程度更高，更不易从源码进行改进。

&emsp; 我们从如下几个方面进行`qemu-3.0.1`源码的改进：

### 1. `cpu-exec.c`文件中`void cpu_exec_step_atomic(CPUState *cpu)`函数调用`start_exclusive()`函数的位置：

&emsp; 首先阐释`start_exclusive()`函数的作用：

 - 通过`atomic_set()`、`atomic_read()`等原子操作函数来读取当前cpu总的运行状态，若有其他cpu正在运行，则将其暂停并保存在一个等待队列中。当且仅当`end_exclusive()`函数被执行时，该cpu才可能被执行。同时，该函数设置了一个锁`mutex`，来保证每个时刻最多只有一个cpu访问`pending_cpus`变量和全部其他cpu的运行状态信息。
 
 &emsp; 下面是`qemu-3.0.1`中`start_exclusive()`函数的实现（位于`cpus-common.c`文件中）：

```
/* Start an exclusive operation.
   Must only be called from outside cpu_exec.  */
void start_exclusive(void)
{
    CPUState *other_cpu;
    int running_cpus;

    qemu_mutex_lock(&qemu_cpu_list_lock);
    exclusive_idle();

    /* Make all other cpus stop executing.  */
    atomic_set(&pending_cpus, 1);

    /* Write pending_cpus before reading other_cpu->running.  */
    smp_mb();
    running_cpus = 0;
    CPU_FOREACH(other_cpu) {
        if (atomic_read(&other_cpu->running)) {
            other_cpu->has_waiter = true;
            running_cpus++;
            qemu_cpu_kick(other_cpu);
        }
    }

    atomic_set(&pending_cpus, running_cpus + 1);
    while (pending_cpus > 1) {
        qemu_cond_wait(&exclusive_cond, &qemu_cpu_list_lock);
    }

    /* Can release mutex, no one will enter another exclusive
     * section until end_exclusive resets pending_cpus to 0.
     */
    qemu_mutex_unlock(&qemu_cpu_list_lock);
}
```

&emsp; 可以看到，`qemu-3.0.1`的`start_exclusive()`函数调用的位置如下：

```
  if (sigsetjmp(cpu->jmp_env, 0) == 0) {
        tb = tb_lookup__cpu_state(cpu, &pc, &cs_base, &flags, cf_mask);
        if (tb == NULL) {
            mmap_lock();
            tb = tb_gen_code(cpu, pc, cs_base, flags, cflags);
            mmap_unlock();
        }

  start_exclusive();
```

&emsp; `sigsetjmp()`函数的作用：

 - 函数定义：int sigsetjmp(sigjmp_buf env, int savesigs)  
 - 函数说明：sigsetjmp()会保存目前堆栈环境，然后将目前的地址作一个记号，而在程序其他地方调用siglongjmp()时便会直接跳到这个记号位置，然后还原堆栈，继续程序的执行。参数env为用来保存目前堆栈环境，一般声明为全局变量；参数savesigs若为非0则代表搁置的信号集合也会一块保存，当sigsetjmp()返回0时代表已经做好记号上，若返回非0则代表由siglongjmp（）跳转回来。  
 - 返回：若直接调用则为0，若从siglongjmp调用返回则为非0 

&emsp; 显然，我们在进行cpu的上下文切换之前，必须要调用`sigsetjmp()`函数将当前的堆栈环境保存，否则这样的跳转是无效的。然而，`qemu-3.1.0`的实现中，无论sigsetjmp返回值是否为0（当为0时代表当前堆栈环境保存成功），都会调用`start_exclusive()`函数开始进行cpu的切换，这显然是不合理的，会造成cpu无效的上下文切换，进而降低cpu的使用效率。因此，我们应该对其做如下改进：

```
  if (sigsetjmp(cpu->jmp_env, 0) == 0) {
        start_exclusive();                                                 // Move to here
        
        tb = tb_lookup__cpu_state(cpu, &pc, &cs_base, &flags, cf_mask);
        if (tb == NULL) {
            mmap_lock();
            tb = tb_gen_code(cpu, pc, cs_base, flags, cflags);
            mmap_unlock();
        }
```

&emsp;

### 2. `cpus.c`文件中`cpu_thread_is_idle(CPUState *cpu)`函数判断依据的加强：

&emsp; 在`qemu-3.0.1`的判断`cpu thread`非空闲的实现中，仅仅使用了`cpu->stop`和一个简单的标志`cpu->queued_work_first`来判断返回值为`false`的条件，需要注意的是`cpu->queued_work_first`所标志的是是否允许当前队列中的cpu优先执行，且单独一个`flag`可能会在其他函数的执行中被错误地修改，导致“死锁“现象的发生：

```
static bool cpu_thread_is_idle(CPUState *cpu)
{
    if (cpu->stop || cpu->queued_work_first) {
        return false;
    }
    // ... ...
 }
```

&emsp; 因此，我们单独创建一个静态内联的函数`static inline bool cpu_work_list_empty(CPUState *cpu)`来检测当前cpu工作队列是否为空，并利用`mutex_lock`锁来保证查询的互斥性。这样，我们避免了错误判断cpu工作队列非空而导致的“死锁”现象的发生，提高了cpu的并行效率。改进如下：

```
static inline bool cpu_work_list_empty(CPUState *cpu)
{
    bool ret;

    qemu_mutex_lock(&cpu->work_mutex);
    ret = QSIMPLEQ_EMPTY(&cpu->work_list);
    qemu_mutex_unlock(&cpu->work_mutex);
    return ret;
}

static bool cpu_thread_is_idle(CPUState *cpu)
{
    if (cpu->stop || !cpu_work_list_empty(cpu)) {
        return false;
    }
    // ... ...
}
```

&emsp;

### 3. `cpu-exec.c`文件中`static inline bool cpu_handle_interrupt(CPUState *cpu, TranslationBlock **last_tb)`函数在debug时`GDB (GNU Debugger)`可能存在的指令丢失问题：

&emsp; `qemu-3.0.1`中`cpu_handle_interrupt()`函数在判断`exit condition`时的部分代码如下：

```
/* The target hook has 3 exit conditions:
False when the interrupt isn't processed,
True when it is, and we should restart on a new TB,
and via longjmp via cpu_loop_exit.  */
else {
    if (cc->cpu_exec_interrupt(cpu, interrupt_request)) {
        replay_interrupt();
        cpu->exception_index = -1;
        *last_tb = NULL;
    }
    /* The target hook may have updated the 'cpu->interrupt_request';
     * reload the 'interrupt_request' value */
    interrupt_request = cpu->interrupt_request;
}
```

&emsp; 在`qemu-3.0.1`的`cpu-exec.c`文件中，`cpu_handle_interrupt()`函数的设计使得当cpu开启`single step`模式时，`GDB (GNU Debugger)`在执行时可能会miss下一条指令，因此我们需要增添一个标志`cpu->singlestep_enabled`，来标志执行该语句时，cpu是否开启了`single step`，以及一个宏`EXCP_DEBUG`来给处于`single step`模式下的`cpu->exception_index`赋值。通过这样的改进，我们较好地避免了`GNB`工作时产生的指令丢失，降低了调试损耗，提高了系统的工作效率。改进如下：

```
/* The target hook has 3 exit conditions:
False when the interrupt isn't processed,
True when it is, and we should restart on a new TB,
and via longjmp via cpu_loop_exit.  */
else {
    if (cc->cpu_exec_interrupt(cpu, interrupt_request)) {
        replay_interrupt();
        /*
        * After processing the interrupt, ensure an EXCP_DEBUG is
        * raised when single-stepping so that GDB doesn't miss the
        * next instruction.
        */
        cpu->exception_index = (cpu->singlestep_enabled ? EXCP_DEBUG : -1);
        *last_tb = NULL;
    }
    /* The target hook may have updated the 'cpu->interrupt_request';
     * reload the 'interrupt_request' value */
    interrupt_request = cpu->interrupt_request;
}
```

&emsp;

-----------------------------




