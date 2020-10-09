# Virsh创建虚拟机

---------------

## 过程中使用的XML文件（注意`emulator`部分）：

```
  <domain type='kvm'>#域类型
    <name>ubuntu-controller</name> #虚拟机的名字，由字母和数字组成，不能包含空格
    <memory unit='GiB'>8</memory> #在不reboot guest的情况下，guset可以使用的最大内存，默认KB为单位
    <currentMemory unit='GiB'>4</currentMemory> #guest启动时内存（当前），可以通过virsh setmem来调整内存，但不能大于最大可使用内存。
    <vcpu>4</vcpu> #分配的虚拟cpu
    <os>
        <type arch='x86_64' machine='pc'>hvm</type> #hvm:全虚拟化
        #<kernel>/tmp/vmlinuz-rhel54</kernel> #kernel：指定guest使用的内核，如果使用ISO（安装时）或guset系统中（系统已经安装完成）的内核，不需要指定该项
        #<initrd>/tmp/initrd-rhel54.img</initrd> #initrd：指定guest使用的ram disk，如果使用ISO（安装时）或guest系统中（系统已经安装完成）的ram disk，不需要指定该项
        #注：kernel 和initrd文件位于RHEL系统光盘的images/pxeboot目录，拷贝这两个文件到本地磁盘，并指定路径。
        #注：这两个元素，如果是为了安装guset而指定，在安装完成以后即可以删除。
        #如果host开启了SELINUX，需要改变文件的security context类型为virt_image_t，从而在启动时libvirtd可以访问这二者
        # chcon -t virt_image_t /tmp/vmlinuz-rhel54
        # chcon -t virt_image_t /tmp/initrd-rhel54.img
        # ls -Z /tmp|grep virt
        <boot dev='hd'/> #boot:指定启动设备，可以重复多行，指定不同的值，作为一个启动设备列表。hd表示从硬盘启动
        <boot dev='cdrom'/> #network表示从pxe启动
    </os>
    <features> #处理器特性
        <acpi/>
        <apic/>
        <pae/>
    </features>
    <clock offset='utc'/> #时钟
    #定义了在kvm环境中power off，reboot，或crash时的默认的动作为destroy。其他允许的动作包括：restart，preserve,rename-restart.
    <on_poweroff>destroy</on_poweroff>
    <on_reboot>restart</on_reboot>
    <on_crash>destroy</on_crash> #destroy：停止该虚拟机。相当于关闭电源
    <devices> #设备定义开始
        <emulator>/usr/bin/qemu-system-x86_64</emulator> #模拟元素，此处写法用于kvm的guest。二进制模拟器设备的完整路径。
        <disk type='file' device='disk'>#disk是用来描述磁盘的主要容器
            <driver name='qemu' type='qcow2'/>
            <source file='/home/zhzej/controller-node.img'/>#指定磁盘上文件的绝对路径
            #使用virtio，采用普通的驱动，即硬盘和网卡都采用默认配置情况下，硬盘是 ide 模式，
            #而网卡工作在 模拟的rtl 8139 网卡下，速度为100M 全双工。
            #采用 virtio 驱动后，网卡工作在 1000M 的模式下，硬盘工作是SCSI模式下。
            #硬盘采用 virtio 后，安装windows 系统，将不能正常的识别硬盘，解决的方法是：
            #从kvm 的官网下载virtio的驱动iso。
            #1. 先采用ide模式安装系统。
            #2. 安装完成后，添加一个virtio模式的硬盘。
            #3. 启动vm后，系统会自动搜索 SCSI的驱动，找到下载的virtio 驱动后，安装即可。
            #4. 修改vm 配置文件，删除掉添加的 vitro 硬盘后，修改ide硬盘为 virtio模式即可
            <target dev='vda' bus='virtio'/>
        </disk>
        <disk type='file' device='cdrom'>
            <source file='/home/iso/ubuntu-14.04.3-server-amd64.iso'/>
            <target dev='hdb' bus='ide'/>
        </disk>
        #使用网桥类型。确保每个kvm guest的mac地址唯一。将创建tun设备，名称为vnetx（x为0,1,2...）
        <interface type='bridge'>
            <source bridge='br0'/>
            <mac address='52:54:02:2B:73:F1'/>
            <model type='virtio'/>
        </interface>
        <interface type='bridge'>
            <source bridge='br0'/>
            <mac address='52:54:02:2B:73:F2'/>
            <model type='virtio'/>
        </interface>
        # 补充：使用默认的虚拟网络代替网桥，即guest为NAT模式。也可以省略mac地址元素，这样将自动生成mac地址。
        # 默认分配192.168.122.x/24的地址，也可以手动指定。网关为192.168.122.1
        #<interface type='network'>
        # <source network='default'/>
        # <mac address="52:54:4a:e1:1c:84"/>
        #</interface>
 
        <input type='mouse' bus='ps2'/> #输入设备
        #定义与guset交互的图形设备。在这个例子中，使用vnc协议。listen的地址为host的地址。prot为-1，表示自动分配端口号。
        <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0' keymap='en-us'/>
    </devices>
</domain>


```
