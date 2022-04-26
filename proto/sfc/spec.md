
# sfc 
注意：这里的描述是我自己的理解,并保留我自己觉得重要的部分，还是得看RFC7655

## 名词解释
https://datatracker.ietf.org/doc/html/rfc7665
### Classification
分类器，类似策略路由，根据报文的字段来决定走哪个SFC
### Classifier
分类器的实例
### sfc(service function chain)
服务功能链定义了一组有序的抽象服务sf，规定了sf的顺序
### sf(service function)
负责对接收到的数据包进行特定处理的功能，抽象服务包括防火墙，WAN，应用加速，DPI（深度包检测），LI(合法拦截)，负载均衡，NAT44,NAT64,NPTv6,HTTP...
### sff(service function forwarder)
sff决定数据包被指向那个sf
### sfp (service function path)
sf路径，决定走按特定的顺序走特定的sf
### sfc Encapsulation:
sfc给报文封装字段，用来给sff,sfp识别报文怎么走，常用的就是nsh(network service header) https://datatracker.ietf.org/doc/html/rfc8300
### sfc proxy
解封/封装 sfc Encapsulation的逻辑组件，比如vpp里面的nsh proxy
### sfc-enabled domain
针对sfc的网络隔离空间，类似vrf
### metadata
提供在分类器和 SF 之间以及 SF 之间交换上下文信息的能力。




## sfc 核心组件
### 简单模型

      o . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
      .  +--------------+                  +------------------~~~
      .  |   Service    |       SFC        |  Service  +---+   +---+
      .  |Classification|  Encapsulation   | Function  |sf1|...|sfn|
   +---->|   Function   |+---------------->|   Path    +---+   +---+
      .  +--------------+                  +------------------~~~
      . SFC-enabled Domain
      o . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

在一个sfc-enabled domain里面，报文从左到右，先经过classifiction匹配策略，然后给报文打上sfc封装，后进入sf集群。



          +----------------+                        +----------------+
          |   SFC-aware    |                        |  SFC-unaware   |
          |Service Function|                        |Service Function|
          +-------+--------+                        +-------+--------+
                  |                                         |
            SFC Encapsulation                       No SFC Encapsulation
                  |                  SFC                    |
     +---------+  +----------------+ Encapsulation     +---------+
     |SFC-Aware|-----------------+  \     +------------|SFC Proxy|
     |    SF   | ... ----------+  \  \   /             +---------+
     +---------+                \  \  \ /
                               +-------+--------+
                               |   SF Forwarder |
                               |      (SFF)     |
                               +-------+--------+
                                       |
                               SFC Encapsulation
                                       |
                           ... SFC-enabled Domain ...
                                       |
                           Network Overlay Transport
                                       |
                                   _,....._
                                ,-'        `-.
                               /              `.
                              |     Network    |
                              `.              /
                                `.__     __,-'
                                    `''''
经过classification后，报文被sfc封装，到达sff，sff到达sfc-unaware的sf时会在sfc proxy解封sfc header，发给sf，当报文回来时，就封装sfc header.
如果是发给sfc-ware的sf的话，就不需要做这种解封封装sfc header的操作。

### sff
SFF 负责使用在 SFC 封装中传递的信息将从网络接收到的数据包和/或帧转发到与给定 SFF 关联的一个或多个 SF，来自 SF 的流量最终返回到同一个 SFF，该 SFF 负责将流量注入网络。
SFF 维护必要的 SFP 转发信息(SFP的实现应该就类似转发表，配置在每个sff下)， sfp转发信息与spi(service path identifier)一一对应，SFF 还具有允许它在应用本地服务功能后将数据包转发到下一个 SFF 的信息.
sff需要完成以下必备的功能：
1. SFP 转发： 当流量从网络到达sff时，SFF 通过 SFC 封装中包含的信息确定应将流量转发到的适当 SF，SF 处理后，流量返回到 SFF，如果需要，将转发到与该 SFF 关联的另一个 SF，如果 SFP 中有另一个非本地（即不同的 SFF）跃点，则 SFF 进一步将流量封装在适当的网络传输协议中，并将其传递给网络，以便沿路径传递给下一个 SFF
2. 终止sfp: 当最后一个 SF 处理完流量后到达 SFF 时，最终 SFF 从服务转发状态知道 SFC 完成,SFF 去掉 SFC 封装，将报文送回网络转发
3. Maintaining flow state: 在某些情况下，SFF 可能是有状态的

### sfc proxy



![image](https://github.com/changluyi/network/blob/master/proto/sfc/A-typical-example-of-service-function-chain-SFC.png)



# nsh
参考

