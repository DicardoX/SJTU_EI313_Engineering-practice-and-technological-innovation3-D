# 利用Python获取kvm虚拟机的监控信息

[参考链接](https://blog.51cto.com/10616534/1878609)

---------------

&emsp; Python脚本代码：

```
  import libvirt
  conn = libvirt.open("qemu:///system")
  for id in conn.listDomainsID():
      domain = conn.lookupByID(id)
      print domain.name()  
      print domain.UUIDString()
      print domain.info()
  conn.close()
```

&emsp; 注意，执行`python check.py`时报错`ImportError: No module named libvirt`。

&emsp; 解决方案：

```
  python3 check.py
```
