vpp 高性能实现：

# 1. 矢量报文减少指令cache miss

## 标量数据报文 

报文时按照到达先后次序来处理，第一个报文处理完，在处理第二个，以此类推。 更为窗体的方式还需要结合处理中断，并遍历调用栈(e.g. calls b、calls c、calls d… return return return),然后从中断返回，函数会频繁嵌套调用。 最后，该过程执行后续三种操作之一 ： a、不处理 ， b、丢弃或重写、c、转发报文。

关于cpu指令可以参考这个
https://www.jianshu.com/p/05c6c1d73144



## 
