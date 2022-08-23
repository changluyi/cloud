
关于dpdk一些关键技术点，还是四年前看的，现在想重新理一下，估计会有不一样的收获。

# uio

linux提供的一种 userspace i/o技术，因为一般linux io驱动都放在kernel层，放在uio后用户空间就能直接调用

## 注册uio设备。

uio源文档https://www.kernel.org/doc/html/v5.0/driver-api/uio-howto.html

摆个经典图uio

![image](https://github.com/changluyi/network/blob/master/dpdk/uio.jpg)

### 注册uio驱动

kernel注册流程首先得有这个驱动，这一步由modprobe uio_pci_generic完成。

modprobe uio_pci_generic把驱动注册在/sys/bus/pci/drivers/uio_pci_generic 中

里面有bind  module  new_id  remove_id  uevent  unbind，modprobe会系统调用init_module，相当于注册驱动后会在用户态提供几种接口，接口以文件形式表现出来。

#### 把设备绑定到uio驱动上

执行dpdk-devbind.py绑定操作后（这个脚本其实也就是写上面几个驱动文件接口），

后会在/dev/uiox上面出现对应的uio设备，uio驱动在内核空间只做了uio设备注册， 和uio内存map。

设备绑定后会生成一些关键目录，/dev/uio0 ，/sys/class/uio/uio0/maps/map0/

1）/dev/uio0 处理设备产生的中断， 有了uio后，可以把大部分响应中断的动作放到用户态，内核在收到设备中断后不做具体事务处理，直接上报用用户态

用户态调用read函数(也可以用select)/dev/uiox去堵塞在这个函数上，一旦设备触发内核中断，会通过read函数立马返回。用户则可以在用户态去自定义中断后的操作。

2） /sys/class/uio/uio0/maps/map0/ ，表示uio设备的内存映射把设备内存映射到进程中，用户态进程可以访问这些目录获取设备内存信息，

然后去直接对这段内存地址读写，那么设备和用户态进程之间就能打通了。

（补一下进程内存分配知识https://www.cnblogs.com/huxiao-tee/p/4660352.html）

启动一个vpp进程，查看/proc/pid/maps可以看到device映射到进程的，fd580000和fdfe0000代表的是文件的起始偏移量

7fea80000000-7fea80020000 rw-s fd580000 00:16 46265                      /sys/devices/pci0000:00/0000:00:11.0/0000:02:05.0/resource0

7fea80020000-7fea80030000 rw-s fdfe0000 00:16 46266                      /sys/devices/pci0000:00/0000:00:11.0/0000:02:05.0/resource2

那么去查看uio设备的地址也是在fd580000和fdfe0000

sys/class/uio/uio0/maps/map0下的

addr name offset size。他们分别是设备映射内存的起始地址, 映射内存的名字，起始地址的页内偏移， 映射内存的大小
 
/sys/class/uio/uio0/maps/map0/addr

0x00000000fd580000

/sys/class/uio/uio0/maps/map1/addr

0x00000000fdfe0000

root@ubuntu:/sys/class/uio/uio0/maps/map0# cat size
0x0000000000020000
root@ubuntu:/sys/class/uio/uio0/maps/map1# cat size
0x0000000000010000，
可以看到分别对应了进程7fea80000000-7fea80020000和7fea80020000-7fea80030000的差值

源码可见rpci_probe=>。。。。=> rte_pci_map_device

注意：这里进程到设备的映射只是控制面的，报文的buffer并不在这解决。

# 内存管理

因为dpdk的卖点是0拷贝，于是dpdk想了个方法，能不能预先去alloc一段内存，让进程和设备都能访问到。设备从物理口收到报文后把包放在这段内存里，应用程序直接去访问这段内存，那不就不需要报文copy了。dpdk就是这么做的，其实这种优化思路很多地方都是这样，比如vhostuser。

其实跟着上面的这个解决思路，我大概会有个问题：网卡怎么和用户态进程去协商这段地址的？














