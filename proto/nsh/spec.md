
# nsh (network service header)
参考https://datatracker.ietf.org/doc/html/rfc8300

NSH组成:
1. Service Function Path identification.
2. Indication of location within a Service Function Path.
3. Optional, per-packet metadata (fixed-length or variable).

## 报文格式：
   The NSH is composed of a 4-byte Base Header, a 4-byte Service Path
   Header, and optional Context Headers, as shown in Figure 2.

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                Base Header                                    |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                Service Path Header                            |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     ~                Context Header(s)                              ~
     |                                                               |

Base Header:  Provides information about the service header and the
      payload protocol.

Service Path Header:  Provides path identification and location
      within a service path.

Context Header:  Carries metadata (i.e., context data) along a
      service path.

### nsh base header

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |Ver|O|U|    TTL    |   Length  |U|U|U|U|MD Type| Next Protocol |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

TTL：Indicates the maximum SFF hops for an SFP

Metadata (MD) Type： 表示强制 NSH Base Header 和 Service Path Header 之外的 NSH 格式,MD Type 定义了所携带的metadata的格式

md_type 0: reserved

md_type 1: 这表示header的格式包含一个Fixed-Length Context Header

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |Ver|O|U|    TTL    |   Length  |U|U|U|U|MD Type| Next Protocol |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |          Service Path Identifier              | Service Index |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     |                 Fixed-Length Context Header                   |
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                          NSH MD Type 0x1

支持 SFC 的 SF 或 SFC 代理需要首先接收数据结构和语义，以便处理放置在context中的数。不知道 MD 类型 1 的 NSH 的context header格式的 SF 或 SFC 代理必须丢弃任何具有此类 NSH 的数据包

md_type 2: 这不强制要求除基本报头和服务路径报头之外的任何报头，但可能包含可选的可变长度上context

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |Ver|O|U|    TTL    |   Length  |U|U|U|U|MD Type| Next Protocol |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |          Service Path Identifier              | Service Index |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     ~              Variable-Length Context Headers  (opt.)          ~
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     
Next Protocol: 表示封装数据的协议类型.

0x1: IPv4
0x2: IPv6
0x3: Ethernet
0x4: NSH
0x5: MPLS
0xFE: Experiment 1
0xFF: Experiment 2

 
可选的可变长度上下文标头必须是 4 字节的整数


      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |          Metadata Class       |      Type     |U|    Length   |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                   Variable-Length Metadata                    |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     
Metadata Class： 定义 Type 字段的范围以提供分层命名空间

        +------------------+------------------------+------------+
        | Value            | Meaning                | Reference  |
        +------------------+------------------------+------------+
        | 0x0000           | IETF Base NSH MD Class | RFC 8300   |
        |                  |                        |            |
        | 0xfff6 to 0xfffe | Experimental           | RFC 8300   |
        |                  |                        |            |
        | 0xffff           | Reserved               | RFC 8300   |
        +------------------+------------------------+------------+
Type：
Lengh: Indicates the length of the variable-length metadata

在接收到属于给定 SFP 的数据包后，如果该数据包中缺少强制处理的上下文报头，则 SFC-ware SF 不得处理该数据包，并且必须在每个 SPI 中至少记录一次错误缺少强制性元数据。

### service path header

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |          Service Path Identifier (SPI)        | Service Index |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

     Service Path Identifier (SPI): 24 bits

     Service Index (SI): 8 bits

#### spi (service path identifier)

唯一标识服务功能路径 (SFP), spi和sfp一一对应。
     
#### si (service index)

提供 SFP 内的位置。给定 SFP 的初始分类器应该将 SI 设置为 255， 在执行所需的服务后，服务功能或 SFC 代理节点必须将服务索引减 1， 新的递减 SI 值必须用于出口数据包的 NSH。

si 与 spi一起用于 SFP 选择和确定路径中的下一个 SFF/SF

其实这个si就和base header ttl很像了，都是经过一跳sff后会减一


## nsh actions

         +-----------+-----------------------+-------+---------------+-------+
         |           | Insert, remove, or    |Forward| Update        |Service|
         |           | replace the NSH       |the NSH| the NSH       |policy |
         |           |                       |packets|               |sel.   |
         |Component  +-------+-------+-------+       +-------+-------+       |
         |           |       |       |       |       |Dec.   |Update |       |
         |           |Insert |Remove |Replace|       |Service|Context|       |
         |           |       |       |       |       |Index  |Header |       |
         +-----------+-------+-------+-------+-------+-------+-------+-------+
         |           |  +    |       |   +   |       |       |   +   |       |
         |Classifier |       |       |       |       |       |       |       |
         +-----------+-------+-------+-------+-------+-------+-------+-------+
         |Service    |       |   +   |       |   +   |       |       |       |
         |Function   |       |       |       |       |       |       |       |
         |Forwarder  |       |       |       |       |       |       |       |
         |(SFF)      |       |       |       |       |       |       |       |
         +-----------+-------+-------+-------+-------+-------+-------+-------+
         |Service    |       |       |       |       |   +   |   +   |   +   |
         |Function   |       |       |       |       |       |       |       |
         |(SF)       |       |       |       |       |       |       |       |
         +-----------+-------+-------+-------+-------+-------+-------+-------+
         |           |  +    |   +   |       |       |   +   |   +   |       |
         |SFC Proxy  |       |       |       |       |       |       |       |
         +-----------+-------+-------+-------+-------+-------+-------+-------+

这里sff的remove action只发生在sfp的最后一跳

NSH Action and Role Mapping

### SFFs and Overlay Selection

SPI 和 SI 的组合提供了逻辑 SF 的标识及其在服务平面内的顺序。该组合用于选择适当的网络定位器进行覆盖转发。逻辑 SF 可以是单个 SF 或一组等效的合格 SF。在后一种情况下，SFF 根据需要在 SF 集合之间提供负载分配。

SPI 到传输封装的映射发生在 SFF 上（如上所述，路径中的第一个 SFF 从分类器获取 NSH 封装的数据包）,其实vpp的nsh map就是sff+classifier的功能。

vppctl create nsh map nsp 370 nsi 254 mapped-nsp 370 mapped-nsi 254 nsh_action pop encap-vxlan4-intf 46

#将spi/si 370/254的报文 replace成 spi/si 370/254，然后从vxlan tunnel封装出去

vppctl create nsh entry nsp 370 nsi 254 md-type 2 next-ethernet (map后的spi/si)

#指定nsh bash header 的md-type和下一跳proto

#### SFF NSH Mapping Example
      +------+------+---------------------+-------------------------+
      | SPI  | SI   | Next Hop(s)         | Transport Encapsulation |
      +------+------+---------------------+-------------------------+
      | 10   | 255  | 192.0.2.1           | VXLAN-gpe               |
      |      |      |                     |                         |
      | 10   | 254  | 198.51.100.10       | GRE                     |
      |      |      |                     |                         |
      | 10   | 251  | 198.51.100.15       | GRE                     |
      |      |      |                     |                         |
      | 40   | 251  | 198.51.100.15       | GRE                     |
      |      |      |                     |                         |
      | 50   | 200  | 01:23:45:67:89:ab   | Ethernet                |
      |      |      |                     |                         |
      | 15   | 212  | Null (end of path)  | None                    |
      +------+------+---------------------+-------------------------+


#### NSH-to-SF Mapping Example
                      +------+-----+----------------+
                      | SPI  | SI  | Next Hop(s)    |
                      +------+-----+----------------+
                      | 10   | 3   | SF2            |
                      |      |     |                |
                      | 245  | 12  | SF34           |
                      |      |     |                |
                      | 40   | 9   | SF9            |
                      +------+-----+----------------+

#### SF Locator Mapping Example

          +------+-------------------+-------------------------+
          | SF   | Next Hop(s)       | Transport Encapsulation |
          +------+-------------------+-------------------------+
          | SF2  | 192.0.2.2         | VXLAN-gpe               |
          |      |                   |                         |
          | SF34 | 198.51.100.34     | UDP                     |
          |      |                   |                         |
          | SF9  | 2001:db8::1       | GRE                     |
          +------+-------------------+-------------------------+
