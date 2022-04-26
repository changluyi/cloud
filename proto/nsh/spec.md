
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


