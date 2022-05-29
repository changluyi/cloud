关于dpdk相当于提供了一个从网卡到用户态的驱动，我觉得从用户态到网卡已经把性能做到极致了。

关于dpdk一些关键技术点，还是三年前看的，现在想重新理一下，估计会有不一样的收获

# uio

linux提供的一种 userspace i/o技术，因为一般linux io驱动都放在kernel层，放在uio后用户空间就能直接调用

### 1 注册uio设备。

uio源文档https://www.kernel.org/doc/html/v5.0/driver-api/uio-howto.html

摆个经典图uio

![image]()

#### 注册uio驱动

kernel注册流程首先得有这个驱动，这一步由modprobe uio_pci_generic完成。

modprobe uio_pci_generic把驱动注册在/sys/bus/pci/drivers/uio_pci_generic 中

里面有bind  module  new_id  remove_id  uevent  unbind，modprobe会系统调用init_module，相当于注册驱动后会在用户态提供几种接口，接口以文件形式表现出来。

#### 把设备绑定到uio驱动上

执行dpdk-devbind.py绑定操作后（这个脚本其实也就是写上面几个驱动文件接口），

后会在/dev/uiox上面出现对应的uio设备，uio驱动在内核空间只做了uio设备注册， 和uio内存map。

设备绑定后会生成一些关键目录，/dev/uio0 ，/sys/class/uio/uio0/maps/map0/

1）/dev/uio0 处理设备产生的中断， 有了uio后，可以把大部分响应中断的动作放到用户态，内核在收到设备中断后不做具体事务处理，直接上报用用户态

用户态调用read/select函数/dev/uiox去堵塞在这个函数上，一旦设备触发内核中断，会通过read函数立马返回。用户则可以在用户态去自定义中断后的操作。

2） /sys/class/uio/uio0/maps/map0/ ，表示uio设备的内存映射把设备内存映射到进程中，为什么要做这个映射，因为进程可以去直接对这段内存地址读写，从而达到操作uio设备的目的

不需要去调用read/write进行系统调用。

（补一下进程内存分配知识https://www.cnblogs.com/huxiao-tee/p/4660352.html）

启动一个vpp进程，查看/proc/pid/maps可以看到device映射到进程的地址是fd580000和fdfe0000

7fea80000000-7fea80020000 rw-s fd580000 00:16 46265                      /sys/devices/pci0000:00/0000:00:11.0/0000:02:05.0/resource0
7fea80020000-7fea80030000 rw-s fdfe0000 00:16 46266                      /sys/devices/pci0000:00/0000:00:11.0/0000:02:05.0/resource2

那么去查看uio的地址
cat /sys/class/uio/uio0/maps/map0/addr
0x00000000fd580000
/sys/class/uio/uio0/maps/map1/addr
0x00000000fdfe0000

源码可见rpci_probe=>。。。。=> rte_pci_map_device

# 内存管理

讲内存管理的时候



