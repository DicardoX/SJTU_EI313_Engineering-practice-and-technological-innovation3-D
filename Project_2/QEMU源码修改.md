# QEMU源码修改：更改QEMU窗口界面标题

---------------------

## 修改QEMU的部分源码

&emsp; 打开qemu文件夹下的`sdl2.c`文件，找到`sdl_update_caption()`函数，将以下部分代码：

```
  if (qemu_name) {
      snprintf(win_title, sizeof(win_title), "QEMU (%s-%d)%s", qemu_name,
             scon->idx, status);
      snprintf(icon_title, sizeof(icon_title), "QEMU (%s)", qemu_name);
  } else {
      snprintf(win_title, sizeof(win_title), "QEMU%s", status);
      snprintf(icon_title, sizeof(icon_title), "QEMU");
  }
```

&emsp; 修改为：

```
  // Modify the name of QEMU virtual machine on the UI windows here
  if (qemu_name) {
      snprintf(win_title, sizeof(win_title), "Chunyu Xue's QEMU (%s-%d)%s", qemu_name,
             scon->idx, status);
      snprintf(icon_title, sizeof(icon_title), "Chunyu Xue's QEMU (%s)", qemu_name);
  } else {
      snprintf(win_title, sizeof(win_title), "Chunyu Xue's QEMU%s", status);
      snprintf(icon_title, sizeof(icon_title), "Chunyu Xue's QEMU");
  }
```

---------------------

## 重新创建未编译环境

&emsp; 将本来编译好的QEMU环境利用VMware的snapshot备份，完成后将qemu下的build文件夹删除，重新创建并进入。运行如下指令进行编译：

```
  ../configure --target-list=x86_64-softmmu --enable-kvm --enable-debug
  make
```





