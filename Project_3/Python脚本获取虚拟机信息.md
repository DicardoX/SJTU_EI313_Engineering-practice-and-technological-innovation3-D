# 利用Python获取kvm虚拟机的监控信息

[参考链接](https://blog.51cto.com/10616534/1878609)

---------------

&emsp; 安装qemu环境：

```
	sudo apt-get install qemu
	sudo apt-get install qemu-kvm
```

&emsp; 安装相关软件和包：

```
	sudo apt-get install virtinst
	sudo apt-get install libvirt-dev
	sudo apt-get install libvirt-bin
```

&emsp; `sudo virt-manager`打开虚拟机管理器，并新建一个虚拟机。

&emsp; `sudo virsh list`可以查看运行过程中的虚拟机；`sudo virsh list --all`可以查看包括关闭状态在内的全部虚拟机。

&emsp; 在虚拟机运行过程中运行如下python脚本代码：

```
	import libvirt
	conn = libvirt.open("qemu:///system")
	for id in conn.listDomainsID():
		domain = conn.lookupByID(id)
	
		print ("The ID of VM: %s" % (domain.UUIDString()))
		print ("The Name of VM: %s" % (domain.name()))  
		print ("The Status of VM: %d (1 for running)" % (domain.info()[0]))
		print ("The Max Memory of VM: %d MB" % (domain.info()[1]))
		print ("The Number of Virtual CPUs: %d" % (domain.info()[3]))
	conn.close()
```

&emsp; 注意，执行`python check.py`时报错`ImportError: No module named libvirt`。

&emsp; 解决方案：

```
	sudo apt-get install python3-libvirt
```

&emsp; 运行python脚本

``` 
	python3 check.py
```

&emsp; 注意，要保证`virt-manager`中的虚拟机处于运行或暂停状态（非关机状态），否则会没有输出。


