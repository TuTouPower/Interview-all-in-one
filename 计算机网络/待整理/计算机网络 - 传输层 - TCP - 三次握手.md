# 计算机网络 - 传输层 - TCP - 三次握手

## 三次握手的准备

+ 初始状态下客户端与服务端皆处于**CLOSED**状态，客户端与服务端分别调用**socket()**创建套接字。

+ 服务端调用**bind()**将服务端的socket绑定到特定的IP。

    > 因为服务端的IP地址是公布出去的，固定的，要交给DNS服务器作为解析域名的结果。所以需要**bind()**。

+ 服务端调用**listen()**进入**LISTEN**状态，使socket监听特定端口。

+ 客户端调用**connect()**，服务端调用**accept()**，这两个函数便是TCP三次握手。

    > 注意：服务端调用accept()，输入监听的socket返回的是另一个与客户端建立连接的socket。这是由于监听socket需要监听大量客户端的连接，不能作为与客户端连接的socket，必须使用一个新的socket代表与客户端的连接。

## 三次握手概述

+ 客户端的 TCP 首先向服务器端的 TCP 发送一个特殊的 TCP 报文段 。
    +  该报文段中**不包含应用层数据** 。
    + 在报文段的首部中的**SYN 比特被置为 1**。 
    + 客户会**随机地选择一个初始序号**（client_isn ），并将此编号放置于该起始的 TCP SYN报文段的**序号字段**中 。 
    + 这个特殊报文段被称为 **SYN 报文段**。
    + 该报文段会被封装在一个 IP 数据报中，并发送给服务器。客户端进入发送报文后进入**SYN_SENT**状态。
+ 一旦包含 TCP SYN 报文段的 IP 数据报到达服务器主机（假定它的确到达了！），服务器会从该数据报中提取出 TCP SYN 报文段，为该 TCP 连接**分配 TCP 缓存和变量**，并向该客户 TCP 发送允许连接的报文段。 （*在完成三次握手的第三步之前分配这些缓存和变量，使得 TCP 易于受到称为 SYN 洪泛的拒绝服务攻击*。 ）
    + 这个允许连接的报文段**也不包含应用层数据** 。 
    + 在报文段的首部中的 **SYN 比特被置为1**。
    + 服务器随机选择自己的**初始序号( server_isn ）**，并将其放置到 TCP 报文段首部的**序号字段**中。
    + 该 TCP 报文段首部的**确认号字段被置为 client_isn + 1** 。
    + 该报文段被称为 **SYNACK 报文段**（ SYN + ACK segment ）。
    + 服务端发送报文后进入**SYN_RCVD**状态。
+ 在收到 SYNACK 报文段后，客户也要给该连接**分配缓存和变量**。 客户主机则向服务器发送另外一个报文段。
    + 该三次握手的第三个阶段可以在报文段负载中**携带客户到服务器的数据**。
    + 该客户通过将值 **server_isn + 1** 放置到 TCP 报文段首部的**确认字段**。
    + 因为连接已经建立了，所以该 **SYN比特被置为 0**。
    + 客户端进行**ESTABLISHED**状态。
+ 服务端收到客户端的ACK报文段后进入**ESTABLISHED**状态。
+ 一旦完成这 3 个步骤，客户和服务器主机就可以相互发送包括数据的报文段了。 <u>在以后每一个报文段中， SYN 比特都将被置为 0。</u> 注意到为了创建该连接，在两台主机之间发送了 3 个分组。 由于这个原因 ， 这种连接创建过程通常被称为 **3 次握手( three- way handshake** ）。
+ ![](D:\Download\桌面\知识体系\图片\三次握手.jpg)

## SYN洪泛攻击

![](D:\Download\桌面\知识体系\图片\SYN.jpg)

![](D:\Download\桌面\知识体系\图片\SYN洪泛攻击2.jpg)

## SYN 攻击

*SYN 攻击*

我们都知道 TCP 连接建立是需要三次握手，假设攻击者短时间伪造不同 IP 地址的 `SYN` 报文，服务端每接收到一个 `SYN` 报文，就进入`SYN_RCVD` 状态，但服务端发送出去的 `ACK + SYN` 报文，无法得到未知 IP 主机的 `ACK` 应答，久而久之就会**占满服务端的 SYN 接收队列（未连接队列）**，使得服务器不能为正常用户服务。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843T2LZtpQAej3iasyuMyvrDvCiazkEndRXrsk2zvcjItPFl6QNvaov3DFw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)SYN 攻击

*避免 SYN 攻击方式一*

其中一种解决方式是通过修改 Linux 内核参数，控制队列大小和当队列满时应做什么处理。

- 当网卡接收数据包的速度大于内核处理的速度时，会有一个队列保存这些数据包。控制该队列的最大值如下参数：

```
net.core.netdev_max_backlog
```



- SYN_RCVD 状态连接的最大个数：

```
net.ipv4.tcp_max_syn_backlog
```



- 超出处理能时，对新的 SYN 直接回 RST，丢弃连接：

```
net.ipv4.tcp_abort_on_overflow
```

*避免 SYN 攻击方式二*

我们先来看下Linux 内核的 `SYN` （未完成连接建立）队列与 `Accpet` （已完成连接建立）队列是如何工作的？

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843ACpyQCkuFKfWROystv5qxMPs71O9FCx9GiaGmeOjCSticOKcX2ZB9p4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)正常流程

正常流程：

- 当服务端接收到客户端的 SYN 报文时，会将其加入到内核的「 SYN 队列」；
- 接着发送 SYN + ACK 给客户端，等待客户端回应 ACK 报文；
- 服务端接收到 ACK 报文后，从「 SYN 队列」移除放入到「 Accept 队列」；
- 应用通过调用 `accpet()` socket 接口，从「 Accept 队列」取出的连接。



![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843AthUXYfJCiakrHFc22ibjA7wmEDrkqwYEsicASXSLfAfSicmFfKicuMLBwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)应用程序过慢

应用程序过慢：

- 如果应用程序过慢时，就会导致「 Accept 队列」被占满。



![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS8432qlbW7sdxgh2Y9E2PtEibl5ORa1Dxt3JhGJTsGpzUtwMxFfuxwnNzPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)受到 SYN 攻击

受到 SYN 攻击：

- 如果不断受到 SYN 攻击，就会导致「 SYN 队列」被占满。

`tcp_syncookies` 的方式可以应对 SYN 攻击的方法：

```
net.ipv4.tcp_syncookies = 1
```



![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843J4ygolhCqYoh52C8EFia9QUGsEY52oGTNicruORWBg0MD7TTeIiajh0rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)tcp_syncookies 应对 SYN 攻击

- 当 「 SYN 队列」满之后，后续服务器收到 SYN 包，不进入「 SYN 队列」；
- 计算出一个 `cookie` 值，再以 SYN + ACK 中的「序列号」返回客户端，
- 服务端接收到客户端的应答报文时，服务器会检查这个 ACK 包的合法性。如果合法，直接放入到「 Accept 队列」。
- 最后应用通过调用 `accpet()` socket 接口，从「 Accept 队列」取出的连接。

## 客户端与服务端状态转移

![](D:\Download\桌面\知识体系\图片\客户端TCP状态转移.jpg)

![](D:\Download\桌面\知识体系\图片\服务端TCP状态转移.jpg)

### 初始序列号（ISN）

> 问：ISN 代表什么？意义何在？ISN 是固定不变的吗？ISN为何要动态随机？

**ISN 是什么？**

答：`ISN` 全称是 `Initial Sequence Number`，是 TCP 发送方的字节数据编号的原点，告诉对方我要开始发送数据的初始化序列号

**ISN 是固定不变的吗？**

答：ISN 如果是固定的，攻击者很容易猜出后续的确认序号，为了安全起见，避免被第三方猜到从而发送伪造的 `RST` 报文，因此 ISN 是动态生成的

## 特殊情况

> 我们来考虑**当一台主机接收到一个 TCP 报文段，其端口号或源 IP 地址与该主机上进行中的套接字都不匹配**的情况 。 例如，假如一台主机接收了具有目的端口 80的一个 TCP SYN 分组，但该主机在端口 80 不接受连接（即它不在端口 80 上运行 Web 服务器） 。 则该主机将向源发送一个特殊重置报文段 。 该 TCP 报文段将 RST 标志位置为 1 。 因此，当主机发送一个重置报文段时，它告诉该源“我没有那个报文段的套接字 。 请不要再发送该报文段了” 。 当一台主机接收一个 UDP 分组，它的目的端口与进行中的 UDP 套接字不匹配，该主机发送一个特殊的 ICMP 数据报。

## 三次握手源码剖析

### connect()

一个 TCP 连接是由客户端发起的，当客户端程序调用 `connect()` 系统调用时，就会与服务端程序建立一个 TCP 连接。`connect()` 系统调用的原型如下：

```c++
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

下面是 `connect()` 系统调用各个参数的作用：

- `sockfd`：由 `socket()` 系统调用创建的文件句柄。
- `addr`：指定要连接的远端 IP 地址和端口。
- `addrlen`：指定参数 `addr` 的长度。

当客户端调用 `connect()` 函数时，会触发内核调用 `sys_connect()` 内核函数，`sys_connect()` 函数实现如下：

```c++
int sys_connect(int fd, struct sockaddr *uservaddr, int addrlen){
	struct socket *sock;
	char address[MAX_SOCK_ADDR];
    int err;
    ...
    // 获取文件句柄对应的socket对象
    sock = sockfd_lookup(fd, &err);
    ...
    // 从用户空间复制要连接的远端IP地址和端口信息
    err = move_addr_to_kernel(uservaddr, addrlen, address);
    ...
    // 调用 inet_stream_connect() 函数完成连接操作
    err = sock->ops->connect(sock, (struct sockaddr *)address, addrlen,sock->file->f_flags);
    ...
    return err;
}
```

`sys_connect()` 内核函数主要完成 3 个步骤：

- 调用 `sockfd_lookup()` 函数获取 `fd` 文件句柄对应的 socket 对象。
- 调用 `move_addr_to_kernel()` 函数从用户空间复制要连接的远端 IP 地址和端口信息。
- 调用 `inet_stream_connect()` 函数完成连接操作。

我们继续分析 `inet_stream_connect()` 函数的实现：

```c++
int inet_stream_connect(struct socket *sock, struct sockaddr * uaddr,                        int addr_len, int flags){
	struct sock *sk = sock->sk;
    int err;
    ...
    if (sock->state == SS_CONNECTING) {
    ...
    } else {
    	// 尝试自动绑定一个本地端口
        if (inet_autobind(sk) != 0)
        	return(-EAGAIN);
        ...
        // 调用 tcp_v4_connect() 进行连接操作
        err = sk->prot->connect(sk, uaddr, addr_len);
        if (err < 0)
        	return(err);
        sock->state = SS_CONNECTING;
    }
    ...
    // 如果 socket 设置了非阻塞, 并且连接还没建立, 那么返回 EINPROGRESS 错误
    if (sk->state != TCP_ESTABLISHED && (flags & O_NONBLOCK))        
    	return (-EINPROGRESS);
    
    // 等待连接过程完成
    if (sk->state == TCP_SYN_SENT || sk->state == TCP_SYN_RECV) {        
    inet_wait_for_connect(sk);
    if (signal_pending(current))
    	return -ERESTARTSYS;
    }
    sock->state = SS_CONNECTED; // 设置socket的状态为connected
    ...
    return(0);
}
```

`inet_stream_connect()` 函数的主要操作有以下几个步骤：

- 调用 `inet_autobind()` 函数尝试自动绑定一个本地端口。
- 调用 `tcp_v4_connect()` 函数进行 TCP 协议的连接操作。
- 如果 socket 设置了非阻塞，并且连接还没建立完成，那么返回 EINPROGRESS 错误。
- 调用 `inet_wait_for_connect()` 函数等待连接服务端操作完成。
- 设置 socket 的状态为 `SS_CONNECTED`，表示连接已经建立完成。

在上面的步骤中，最重要的是调用 `tcp_v4_connect()` 函数进行连接操作，我们来分析一下 `tcp_v4_connect()` 函数的实现：

```c++
int tcp_v4_connect(struct sock *sk, struct sockaddr *uaddr, int addr_len){    
	struct tcp_opt *tp = &(sk->tp_pinfo.af_tcp);
    struct sockaddr_in *usin = (struct sockaddr_in *)uaddr;
    struct sk_buff *buff;
    struct rtable *rt;
    u32 daddr, nexthop;
    int tmp;
    ...
    nexthop = daddr = usin->sin_addr.s_addr;
    ...
    // 1. 获取发送数据的路由信息
    tmp = ip_route_connect(&rt, nexthop, sk->saddr,					
    						RT_TOS(sk->ip_tos)|RTO_CONN|sk->localroute,
    						sk->bound_dev_if);
    ...
    dst_release(xchg(&sk->dst_cache, rt)); // 2. 设置sk的路由信息
    
    // 3. 申请一个skb数据包对象
    buff = sock_wmalloc(sk, (MAX_HEADER + sk->prot->max_header), 0, GFP_KERNEL);
    ...
    sk->dport = usin->sin_port; // 4. 设置目的端口
    sk->daddr = rt->rt_dst;     // 5. 设置目的IP地址
    ...
    if (!sk->saddr)
    	sk->saddr = rt->rt_src; // 6. 如果没有指定源IP地址, 那么使用路由信息的源IP地址
    sk->rcv_saddr = sk->saddr;
    ...
    // 7. 初始化TCP序列号
    tp->write_seq = secure_tcp_sequence_number(sk->saddr, sk->daddr, sk->sport,                                               usin->sin_port);
    ...
    // 8. 重置TCP最大报文段大小
    tp->mss_clamp = ~0;
    ...
    // 9. 调用 tcp_connect() 函数继续进行连接操作
    tcp_connect(sk, buff, rt->u.dst.pmtu);
    return 0;
}
```

`tcp_v4_connect()` 函数只是做一些连接前的准备工作，如下：

- 调用 `ip_route_connect()` 函数获取发送数据的路由信息，并且将路由信息保存到 socket 对象的路由缓存中。
- 调用 `sock_wmalloc()` 函数申请一个 skb 数据包对象。
- 设置 `目的端口` 和 `目的 IP 地址`。
- 如果没有指定 `源 IP 地址`，那么使用路由信息中的 `源 IP 地址`。
- 调用 `secure_tcp_sequence_number()` 函数初始化 TCP 序列号。
- 重置 TCP 协议最大报文段的大小。
- 调用 `tcp_connect()` 函数发送 `SYN包` 给服务端程序。

由于 `TCP三次握手` 的第一步是由客户端发送 `SYN包` 给服务端，所以我们主要关注 `tcp_connect()` 函数的实现，其代码如下：

```
void tcp_connect(struct sock *sk, struct sk_buff *buff, int mtu)
{
    struct dst_entry *dst = sk->dst_cache;
    struct tcp_opt *tp = &(sk->tp_pinfo.af_tcp);

    skb_reserve(buff, MAX_HEADER + sk->prot->max_header); // 保留所有的协议头部空间

    tp->snd_wnd = 0;
    tp->snd_wl1 = 0;
    tp->snd_wl2 = tp->write_seq;
    tp->snd_una = tp->write_seq;
    tp->rcv_nxt = 0;
    sk->err = 0;
    // 设置TCP头部长度
    tp->tcp_header_len = sizeof(struct tcphdr) +
                           (sysctl_tcp_timestamps ? TCPOLEN_TSTAMP_ALIGNED : 0);
    ...
    tcp_sync_mss(sk, mtu); // 设置TCP报文段最大长度
    ...
    TCP_SKB_CB(buff)->flags = TCPCB_FLAG_SYN; // 设置SYN标志为1(表示这是一个SYN包)
    TCP_SKB_CB(buff)->sacked = 0;
    TCP_SKB_CB(buff)->urg_ptr = 0;
    buff->csum = 0;
    TCP_SKB_CB(buff)->seq = tp->write_seq++;   // 设置序列号
    TCP_SKB_CB(buff)->end_seq = tp->write_seq; // 设置确认号
    tp->snd_nxt = TCP_SKB_CB(buff)->end_seq;

    // 初始化滑动窗口的大小
    tp->window_clamp = dst->window;
    tcp_select_initial_window(sock_rspace(sk)/2, tp->mss_clamp,
                              &tp->rcv_wnd, &tp->window_clamp,
                              sysctl_tcp_window_scaling, &tp->rcv_wscale);
    ...
    tcp_set_state(sk, TCP_SYN_SENT); // 设置 socket 的状态为 SYN_SENT

    // 调用 tcp_v4_hash() 函数把 socket 添加到 tcp_established_hash 哈希表中
    sk->prot->hash(sk);

    tp->rto = dst->rtt;
    tcp_init_xmit_timers(sk); // 设置超时重传定时器
    ...
    // 把 skb 添加到 write_queue 队列中, 用于重传时使用
    __skb_queue_tail(&sk->write_queue, buff);
    TCP_SKB_CB(buff)->when = jiffies;
    ...
    // 调用 tcp_transmit_skb() 函数构建 SYN 包发送给服务端程序
    tcp_transmit_skb(sk, skb_clone(buff, GFP_KERNEL));
    ...
}
```

`tcp_connect()` 函数的实现虽然比较长，但是逻辑相对简单，就是设置 TCP 头部各个字段的值，然后把数据包发送给服务端。下面列出 `tcp_connect()` 函数主要的工作：

- 设置 TCP 头部的 `SYN 标志位` 为 1 (表示这是一个 `SYN包`)。
- 设置 TCP 头部的序列号和确认号。
- 初始化滑动窗口的大小。
- 设置 socket 的状态为 `SYN_SENT`，可参考上面三次握手的状态图。
- 调用 `tcp_v4_hash()` 函数把 socket 添加到 `tcp_established_hash` 哈希表中，用于通过 IP 地址和端口快速查找到对应的 socket 对象。
- 设置超时重传定时器。
- 把 skb 添加到 `write_queue` 队列中, 用于超时重传。
- 调用 `tcp_transmit_skb()` 函数构建 `SYN包` 发送给服务端程序。

> **注意：**Linux 内核通过 `tcp_established_hash` 哈希表来保存所有的 TCP 连接 socket 对象，而哈希表的键值就是连接的 IP 和端口，所以可以通过连接的 IP 和端口从 `tcp_established_hash` 哈希表中快速找到对应的 socket 连接。如下图所示：
>
> ![图片](https://mmbiz.qpic.cn/mmbiz_png/ciab8jTiab9J7Jb9R1hZMTYvkyp7uJ0DyhAf0YuLg8AMRTOYerRFVyiaKRTVtDltIGHGicCibFehl5yyf4F7TicwcSkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



通过上面的分析，构建 `SYN包` 并且发送给服务端是通过 `tcp_transmit_skb()` 函数完成的，所以我们来分析一下 `tcp_transmit_skb()` 函数的实现：

```c++
void tcp_transmit_skb(struct sock *sk, struct sk_buff *skb)
{
    if (skb != NULL) {
        struct tcp_opt *tp = &(sk->tp_pinfo.af_tcp);
        struct tcp_skb_cb *tcb = TCP_SKB_CB(skb);
        int tcp_header_size = tp->tcp_header_len;
        struct tcphdr *th;
        ...
        // TCP头部指针
        th = (struct tcphdr *)skb_push(skb, tcp_header_size);
        skb->h.th = th;

        skb_set_owner_w(skb, sk);

        // 构建 TCP 协议头部
        th->source = sk->sport;                // 源端口
        th->dest = sk->dport;                  // 目标端口
        th->seq = htonl(TCP_SKB_CB(skb)->seq); // 请求序列号
        th->ack_seq = htonl(tp->rcv_nxt);      // 应答序列号
        th->doff = (tcp_header_size >> 2);     // 头部长度
        th->res1 = 0;
        *(((__u8 *)th) + 13) = tcb->flags;     // 设置TCP头部的标志位

        if (!(tcb->flags & TCPCB_FLAG_SYN))
            th->window = htons(tcp_select_window(sk)); // 滑动窗口大小

        th->check = 0;                                 // 校验和
        th->urg_ptr = ntohs(tcb->urg_ptr);             // 紧急指针
        ...
        // 计算TCP头部的校验和
        tp->af_specific->send_check(sk, th, skb->len, skb);
        ...
        tp->af_specific->queue_xmit(skb); // 调用 ip_queue_xmit() 函数发送数据包
    }
}
```

`tcp_transmit_skb()` 函数的实现相对简单，就是构建 TCP 协议头部，然后调用 `ip_queue_xmit()` 函数将数据包交由 IP 协议发送出去。

至此，客户端就发送了一个 `SYN包` 给服务端，也就是说，`TCP 三次握手` 的第一步已经完成。

### accept()