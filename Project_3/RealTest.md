# 利用Python获取kvm虚拟机的监控信息

[参考链接](https://blog.51cto.com/10616534/1878609)

---------------

&emsp; Python脚本代码：

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
