# TCP 状态转换

<!-- vim-markdown-toc GFM -->

* [状态转换（11 个状态）](#状态转换11-个状态)
    * [`CLOSED`](#closed)
    * [`LISTEN`](#listen)
    * [`SYN_SENT`](#syn_sent)
    * [`SYN_RCVD`](#syn_rcvd)
    * [`ESTABLISHED`](#established)
    * [`FIN_WAIT_1`](#fin_wait_1)
    * [`CLOSE_WAIT`](#close_wait)
    * [`FIN_WAIT_2`](#fin_wait_2)
    * [`LAST_ACK`](#last_ack)
    * [`TIME_WAIT`](#time_wait)
    * [`CLOSING`](#closing)
* [疑问](#疑问)
    * [两军问题和拜占庭将军问题？](#两军问题和拜占庭将军问题)
    * [为什么要三次握手？为什么要四次挥手？](#为什么要三次握手为什么要四次挥手)
    * [MSL 时间是多久？](#msl-时间是多久)
    * [为什么需要 `TIME_WAIT` 状态？为什么是 2MSL 时间？](#为什么需要-time_wait-状态为什么是-2msl-时间)
    * [存在 `TIME_WAIT` 状态过多的危害和处理办法？](#存在-time_wait-状态过多的危害和处理办法)
    * [存在 `CLOSE_WAIT` 状态过多的原因？](#存在-close_wait-状态过多的原因)
    * [存在 `FIN_WAIT_2` 状态过多的处理方法？](#存在-fin_wait_2-状态过多的处理方法)
    * [主动关闭的一方](#主动关闭的一方)
    * [支持的最大 TCP 连接数](#支持的最大-tcp-连接数)
* [参考资料](#参考资料)

<!-- vim-markdown-toc -->

## 状态转换（11 个状态）

```
https://tools.ietf.org/html/rfc793
TCP Connection State Diagram


                              +---------+ ---------\      active OPEN
                              |  CLOSED |            \    -----------
                              +---------+<---------\   \   create TCB
                                |     ^              \   \  snd SYN
                   passive OPEN |     |   CLOSE        \   \
                   ------------ |     | ----------       \   \
                    create TCB  |     | delete TCB         \   \
                                V     |                      \   \
                              +---------+            CLOSE    |    \
                              |  LISTEN |          ---------- |     |
                              +---------+          delete TCB |     |
                   rcv SYN      |     |     SEND              |     |
                  -----------   |     |    -------            |     V
 +---------+      snd SYN,ACK  /       \   snd SYN          +---------+
 |         |<-----------------           ------------------>|         |
 |   SYN   |                    rcv SYN                     |   SYN   |
 |   RCVD  |<-----------------------------------------------|   SENT  |
 |         |                    snd ACK                     |         |
 |         |------------------           -------------------|         |
 +---------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +---------+
   |           --------------   |     |   -----------
   |                  x         |     |     snd ACK
   |                            V     V
   |  CLOSE                   +---------+
   | -------                  |  ESTAB  |
   | snd FIN                  +---------+
   |                   CLOSE    |     |    rcv FIN
   V                  -------   |     |    -------
 +---------+          snd FIN  /       \   snd ACK          +---------+
 |  FIN    |<-----------------           ------------------>|  CLOSE  |
 | WAIT-1  |------------------                              |   WAIT  |
 +---------+          rcv FIN  \                            +---------+
   | rcv ACK of FIN   -------   |                            CLOSE  |
   | --------------   snd ACK   |                           ------- |
   V        x                   V                           snd FIN V
 +---------+                  +---------+                   +---------+
 |FINWAIT-2|                  | CLOSING |                   | LAST-ACK|
 +---------+                  +---------+                   +---------+
   |                rcv ACK of FIN |                 rcv ACK of FIN |
   |  rcv FIN       -------------- |    Timeout=2MSL -------------- |
   |  -------              x       V    ------------        x       V
    \ snd ACK                 +---------+delete TCB         +---------+
     ------------------------>|TIME WAIT|------------------>| CLOSED  |
                              +---------+                   +---------+

```

### `CLOSED`

`CLOSED` 表示初始状态。

### `LISTEN`

`LISTEN` 表示服务器端的某个 SOCKET 处于监听状态，可以接受连接了。

### `SYN_SENT`

主动打开的一方称为客户端，客户端端口号一般随机分配。

`SYN_SENT` 状态表示客户端已发送 SYN 报文。

这个状态是客户端的 SOCKET 在建立 TCP 连接时的三次握手会话过程中的一个中间状态。

### `SYN_RCVD`

被动打开的一方称为服务端，服务端端口号一般固定。

`SYN_RCVD` 状态表示服务端已收到 SYN 报文。并发出 SYN,ACK 报文。

这个状态是服务端的 SOCKET 在建立 TCP 连接时的三次握手会话过程中的一个中间状态。

### `ESTABLISHED`

`ESTABLISHED` 表示连接已经建立了。

三次握手过程中，处于 `SYN_SENT` 状态的客户端收到服务端的 SYN,ACK 报文后，发出 ACK 报文，然后进入 `ESTABLISHED` 状态。

三次握手过程中，处于 `SYN_RCVD` 状态的服务端收到客户端的 ACK 报文后，进入 `ESTABLISHED` 状态。

### `FIN_WAIT_1`

主动关闭的一方，发出 FIN 报文后，从 `ESTABLISHED` 状态进入 `FIN_WAIT_1` 状态。

### `CLOSE_WAIT`

被动关闭的一方，收到 FIN 报文后，发出 ACK 报文，从 `ESTABLISHED` 状态进入 `CLOSE_WAIT` 状态。

收到一个 FIN 报文只意味着这一方向上不会再有数据流入，被动关闭的一方仍能发送数据。

### `FIN_WAIT_2`

主动关闭的一方，收到 ACK 报文后，从 `FIN_WAIT_1` 状态进入 `FIN_WAIT_2` 状态。

### `LAST_ACK`

被动关闭的一方，发送完所有数据后，发出 FIN 报文，从 `CLOSE_WAIT` 状态进入 `LAST_ACK` 状态。

收到 ACK 报文后，从 `LAST_ACK` 状态进入 `CLOSED` 状态。

### `TIME_WAIT`

主动关闭的一方，收到 FIN 报文后，发出 ACK 报文，从 `FIN_WAIT_2` 状态进入 `TIME_WAIT` 状态。

经过 2MSL 时间周期没有再收到另一方的 FIN 报文之后，从 `TIME_WAIT` 状态进入 `CLOSED` 状态。

### `CLOSING`

主动关闭的一方，如果发出 FIN 报文后，同时也收到了 FIN 报文，则发出 ACK 报文，从 `ESTABLISHED` 状态进入 `CLOSING` 状态。

这种状态比较特殊，实际情况中很少见，只有在双方几乎同时 CLOSE 一个 SOCKET 的时候，才会出现双方同时发送 FIN 报文的情况。

收到 ACK 报文后，从 `CLOSING` 状态直接进入 `TIME_WAIT` 状态。

## 疑问

### 两军问题和拜占庭将军问题？

两军问题的根本问题在于信道的不可靠。

即使是三次握手，也并不能够彻底解决两军问题，只是在现实成本可控的条件下，我们把 TCP 协议当作了两军问题的现实可解方法。

拜占庭将军问题假设信道一定是可靠的，并在这个前提下，去做一致性和容错性相关研究。

### 为什么要三次握手？为什么要四次挥手？

TCP 是全双工通讯协议

三次握手原因：确认双方的接收与发送能力是正常的，作为被动打开的一方要想初始化发送通道，必须也进行一轮 SYN + ACK。

四次挥手原因：每个方向必须单独关闭，保证服务器与客户端都能完全的接受对方发送的数据。

### MSL 时间是多久？

MSL 是指 Max Segment Lifetime，即数据包在网络中的最大生存时间。

每种 TCP 协议的实现方法均要指定一个合适的 MSL 值，如 RFC1122 给出的建议值为 2 分钟，
又如 Berkeley 体系的 TCP 实现通常选择 30 秒作为 MSL 值。这意味着 TIME_WAIT 的典型持续时间为 1-4 分钟。

### 为什么需要 `TIME_WAIT` 状态？为什么是 2MSL 时间？

1.  为实现 TCP 全双工连接的可靠释放

    假设主动关闭的一方最后发送的 ACK 报文在网络中丢失，由于 TCP 协议的重传机制，
    被动关闭的一方将会重发 FIN 报文，主动关闭的一方再次收到 FIN 报文后需要继续发送 ACK 报文。

    如果主动关闭一方不维护这样一个 `TIME_WAIT` 状态，那么当被动关闭的一方重发的 FIN 报文到达时，
    主动关闭的一方的 TCP 传输层会用 RST 包响应，这会被对方认为是有错误发生，然而事实上这只是正常的关闭连接过程，并非异常。

2.  使旧的数据包在网络中因过期而消失

    如果主动关闭一方不维护这样一个 `TIME_WAIT` 状态，因某些原因，我们先关闭，接着很快以相同的四元组建立一条新连接。
    TCP 协议栈是无法区分前后两条 TCP 连接的不同的，会导致前一条连接的数据会被新创建的连接认为是正常数据向上传递到应用层。
    从而引起数据错乱进而导致各种无法预知的诡异现象。

    由于 TIME_WAIT 状态持续时间为 2MSL，这样保证了旧 TCP 连接双工链路中的旧数据包均因过期（超过 MSL）而消失，
    此后，就可以用相同的四元组建立一条新连接而不会发生前后两次连接数据错乱的情况。

### 存在 `TIME_WAIT` 状态过多的危害和处理办法？

1.  若主动关闭方是主动打开方

    会占用大量随机端口号，如果端口号耗尽会导致：`cannot assign requested address`

    调整内核参数： [Kernel network parameters](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)

    ```bash
    # 修改此本地端口范围限制
	# Defines the local port range that is used by TCP and UDP to
	# choose the local port. The first number is the first, the
	# second the last local port number.
	# If possible, it is better these numbers have different parity
	# (one even and one odd value).
	# Must be greater than or equal to ip_unprivileged_port_start.
	# The default values are 32768 and 60999 respectively.
    net.ipv4.ip_local_port_range = 1024 60999

    # Enable timestamps as defined in RFC1323
    net.ipv4.tcp_timestamps = 1
    # Enable reuse of TIME-WAIT sockets for new connections when it is safe from protocol viewpoint.
    net.ipv4.tcp_tw_reuse = 1

    # 启用 TIME-WAIT 状态 sockets 的快速回收，这个选项不推荐启用。
    # 在 NAT(Network Address Translation) 网络下，会导致大量的 TCP 连接建立错误。
    # Enable fast recycling of TIME-WAIT sockets. Enabling this option is not recommended since this causes
    # problems when working with NAT (Network Address Translation).
    net.ipv4.tcp_tw_recycle = 0

    # 调小这个值也可以触发 TIME_WAIT 销毁，这个选项不推荐使用。
    # 因为这个值一般用于防止 Dos 攻击，一般都是调大而不要调小
    # 比如加内存后可以适当调大一些，避免 `time wait bucket table overflow`
	# Maximal number of timewait sockets held by system simultaneously.
	# If this number is exceeded time-wait socket is immediately destroyed
	# and warning is printed. This limit exists only to prevent
	# simple DoS attacks, you _must_ not lower the limit artificially,
	# but rather increase it (probably, after increasing installed memory),
	# if network conditions require more than default value.
    net.ipv4.tcp_max_tw_buckets = 300000
    ```

2.  若主动关闭方是被动打开方

    虽然端口号固定，但是每一个 TCP 连接都要占一个文件描述符，如果文件描述符耗尽导致：`too many open files`

    修改文件句柄限制：`ulimit` 命令、软限制、硬限制等。

    提示：这种情况下启用 `net.ipv4.tw_reuse` 并没有任何卵用。

    ```bash
    # 系统最大文件打开数
    fs.file-max = 20000000

    # 单个用户进程最大文件打开数
    fs.nr_open = 20000000
    ```

3.  过多的话是否会会占用内存？

    TIME-WAIT状态的连接占用内存非常的小，简直可以无视。参考：https://bit.ly/3iL28cv

    对于接收缓冲区，只有当有数据可读但应用程序尚未读取的时候才占内存。

    对于发送缓冲区，只有等待发送的数据和发送之后尚未收到 ACK 的数据才占用内存，
    在稳态下，发送缓冲区占用的内存等于 BDP（Bandwidth-delay product）。
    比如考虑发送方每秒钟发 1MB 数据，rtt 是 50ms 的情况，那么发送缓冲区平均占用 51.2 KB （不计 skbuff 的额外开销）。

    ```bash
    # 接收缓冲区
    # tcp_rmem - vector of 3 INTEGERs: min, default, max
    #     min: Minimal size of receive buffer used by TCP sockets.
    #     It is guaranteed to each TCP socket, even under moderate memory
    #     pressure.
    #     Default: 4K
    #
    #     default: initial size of receive buffer used by TCP sockets.
    #     This value overrides net.core.rmem_default used by other protocols.
    #     Default: 87380 bytes. This value results in window of 65535 with
    #     default setting of tcp_adv_win_scale and tcp_app_win:0 and a bit
    #     less for default tcp_app_win. See below about these variables.
    #
    #     max: maximal size of receive buffer allowed for automatically
    #     selected receiver buffers for TCP socket. This value does not override
    #     net.core.rmem_max.  Calling setsockopt() with SO_RCVBUF disables
    #     automatic tuning of that socket's receive buffer size, in which
    #     case this value is ignored.
    #     Default: between 87380B and 6MB, depending on RAM size.
    net.ipv4.tcp_rmem = 4096        87380   6291456

    # 发送缓冲区
    # tcp_wmem - vector of 3 INTEGERs: min, default, max
    #     min: Amount of memory reserved for send buffers for TCP sockets.
    #     Each TCP socket has rights to use it due to fact of its birth.
    #     Default: 4K
    #
    #     default: initial size of send buffer used by TCP sockets.  This
    #     value overrides net.core.wmem_default used by other protocols.
    #     It is usually lower than net.core.wmem_default.
    #     Default: 16K
    #
    #     max: Maximal amount of memory allowed for automatically tuned
    #     send buffers for TCP sockets. This value does not override
    #     net.core.wmem_max.  Calling setsockopt() with SO_SNDBUF disables
    #     automatic tuning of that socket's send buffer size, in which case
    #     this value is ignored.
    #     Default: between 64K and 4MB, depending on RAM size.
    net.ipv4.tcp_wmem = 4096        16384   4194304
    ```

### 存在 `CLOSE_WAIT` 状态过多的原因？

被动关闭的一方没有去 CLOSE，一般都是程序代码问题。

### 存在 `FIN_WAIT_2` 状态过多的处理方法？

```bash
 # 调整 TIME_WAIT_2 到 TIME_WAIT 的超时时间，默认是 60s，优化到 30s
 # The length of time an orphaned (no longer referenced by any
 # application) connection will remain in the FIN_WAIT_2 state
 # before it is aborted at the local end.  While a perfectly
 # valid "receive only" state for an un-orphaned connection, an
 # orphaned connection in FIN_WAIT_2 state could otherwise wait
 # forever for the remote to close its end of the connection.
 # Cf. tcp_max_orphans
 net.ipv4.tcp_fin_timeout = 30
```

### 主动关闭的一方

*   对于基于 TCP 的 HTTP 协议，一般由服务端主动关闭连接，即主动关闭的一方的是被动打开方。

    因此对于访问量大的 Web Server，会存在大量的 `TIME_WAIT` 状态。

    HTTP 协议 1.1 版规定 default 行为是 Keep-Alive，可以重用 TCP 连接传输多个 request/response。

*   对于后端程序作为客户端去操作数据库，一般由客户端主动关闭连接，即主动关闭的一方是主动打开方。

### 支持的最大 TCP 连接数

*   客户端最大 TCP 连接数

    客户端每次发起 TCP 连接请求时，若没有绑定端口，通常会让系统随机选取一个空闲的本地端口。

    不考虑其他因素的极端情况下（例如端口号 1024 以下是系统保留），因此最大 TCP 连接数为 65535。

*   服务端最大 TCP 连接数

    服务端通常固定在某个本地端口上监听。

    不考虑其他因素的极端情况下，最大 TCP 连接数为：IP 数 x 端口数，即 2 的 32 次方 x 2 的 16 次方 = 2 的 48 次方

    然而由于每一个 TCP 连接都要占一个文件描述符，连接数还受到文件描述符数量限制。

## 参考资料

*   [RFC 793 - Transmission Control Protocol](https://tools.ietf.org/html/rfc793)
*   [TCP 状态转换图](https://developer.aliyun.com/article/338169)
*   [两军问题与拜占庭将军问题](https://www.cnblogs.com/charlesblc/p/6271472.html)
*   [TCP/IP 详解 -- TIME_WAIT 状态存在的原因](https://blog.csdn.net/yusiguyuan/article/details/38984759)
*   [再叙 TIME_WAIT](https://blog.huoding.com/2013/12/31/316)
*   [Linux 中每个 TCP 连接最少占用多少内存？](https://zhuanlan.zhihu.com/p/25241630)
*   [不要在linux上启用net.ipv4.tcp_tw_recycle参数 – CFC4N的博客](https://www.cnxct.com/coping-with-the-tcp-time_wait-state-on-busy-linux-servers-in-chinese-and-dont-enable-tcp_tw_recycle/)
*   [［从 0 到 1 编写服务器］TCP 连接建立与断开状态变化](https://www.hoohack.me/2018/09/27/webser-zero-to-hero-tcp-status)

