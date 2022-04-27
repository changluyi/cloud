
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
     




