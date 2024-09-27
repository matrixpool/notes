=====================
Kernel TLS Offload
=====================

.. image:: _static/tls-offload-layers.svg
    :alt: KTLS framework


- openEuler 支持KTLS SM4-GCM

- strparser

KTLS call flow
-------------------

- TX
    do_tls_setsockopt_conf->tls_set_device_offload---->tls_sw_fallback_init->crypto_alloc_aead
                                                    |
                                                    | --->tls_set_sw_offload->crypto_alloc_aead 

- RX
    do_tls_setsockopt_conf->tls_set_device_offload_rx->tls_set_sw_offload->crypto_alloc_aead

load KTLS
--------------------
Tongsuo(ktls_enable) -> setsockopt->tcp_setsockopt->tcp_set_ulp->tls_init

openssl使用SSL_write进行发包，是怎么调到KTLS模块在内核层进行数据加密封装的？
openssl作为客户端与服务端建立连接后，需要使用SSL_set_fd把SSL对象与fd进行绑定，此时会调用ktls_enable函数，
ktls_enable调用

.. code-block:: c

    setsockopt(fd, SOL_TCP, TCP_ULP, "tls", sizeof("tls"))

内核中的tcp_setsockopt会收到请求，根据 ``TCP_ULP`` 和 ``tls`` 调用ULP模块的tcp_set_ulp函数初始化ktls模块，
ktls模块的tls_init模块首先会创建该sock的tls_context上下文并指向 ``icsk_ulp_data`` 。然后把struct sock的struct proto更改为KTLS的proto（原来指向tcp_proto）。


tcp协议栈对struct sock sk有两把锁，第一把是sk_lock.slock，第二把则是sk_lock.owned。sk_lock.slock用于获取struct sock sk对象的成员的修改权限；sk_lock.owned用于区分当前是进程上下文或是软中断上下文，为进程上下文时sk_lock.owned会被置1，中断上下文为0。

如果是要对sk修改，首先是必须拿锁sk_lock.slock，其后是判断当前是软中断或是进程上下文，如果是进程上下文，那么一般也不能修改sk

tcp_inq

1、struct socket、struct sock和struct inet_sock的关系和区别
socket是内核抽象出的一个通用结构体，主要是设置了一些跟fs相关的字段，而真正跟网络通信相关的字段结构体是struct sock。
struct sock是网络层对于struct socket的表示，其中成员非常多，这里只介绍其中一部分。
struct sock 是一个通用的、协议无关的套接字数据结构，是 Linux 内核网络栈中套接字的核心表示。它用于存储与一个网络连接相关的各种信息，如连接状态、缓冲区、定时器、协议特定的处理函数等。

功能：
struct sock 包含了所有套接字（socket）的通用信息，适用于所有网络协议（如 TCP、UDP、RAW 等）。其中包含了如状态（sk_state）、发送和接收缓冲区（sk_send_head、sk_receive_queue）、网络协议操作（sk_prot）、地址族（sk_family）等字段。
应用场景：
由于它是通用的，所有协议的套接字实现（如 TCP 套接字、UDP 套接字等）都基于 struct sock。协议栈中的各层操作主要通过 struct sock 指针来进行操作。

struct inet_sock 是 struct sock 的一个子结构，它通过包含 struct sock 结构体或其指针来继承其所有的通用属性和方法，同时添加了特定于 IPv4 协议的扩展属性和方法。因此，在使用 inet_sock 时，可以方便地通过强制类型转换将其作为通用的 sock 类型来使用，这样便实现了对 IPv4 和其他协议的兼容性。

2、sk_wq如何工作的
3、sock_def_readable
4、inet_sendmsg和tcp_sendmsg的关系和区别
inet_sendmsg 是一个更高层次的函数，用于处理来自用户空间的数据发送请求，并将请求转发给适当的传输层协议（如 TCP 的 tcp_sendmsg）。tcp_sendmsg 则是一个特定于 TCP 协议的函数，负责实现 TCP 连接上的数据发送逻辑。两者相互配合，共同实现 Linux 内核中的数据发送功能。
5、sk_wait_data

kthreadd是kworker、ksoftirqd的父线程。kthreadd 和 ksoftirqd 是 Linux 内核中两个不同的内核线程，分别负责线程管理和软中断处理。kthreadd 是所有内核线程的管理者，而 ksoftirqd 是特定用于处理软中断的线程。两者之间没有直接的功能重叠，但 kthreadd 是 ksoftirqd 的创建者和管理者。

sock_owned_by_user_nocheck 是 Linux 内核中一个用于检查套接字（socket）是否被用户持有的函数。它的作用是判断当前套接字是否已被用户空间进程锁定，从而避免可能的竞争条件或数据不一致。

sock_owned_by_user_nocheck 函数的作用
sock_owned_by_user_nocheck 函数的主要作用是快速检查一个套接字是否被用户（进程）持有（即被 lock_sock 锁住），但它不执行内核中的额外检查。这通常用于一些对性能要求高的地方，避免因多余的检查而带来的性能损失。


TCP_INQ 是 Linux 内核中的一个套接字选项，它用于获取有关 TCP 套接字接收缓冲区中可读取数据量的信息。这是在 Linux 内核 4.18 中引入的一个选项，通常用于优化应用程序的接收数据操作，例如在高性能服务器或网络应用程序中。

TCP_INQ 的作用
主要功能：TCP_INQ 选项允许应用程序通过设置一个套接字选项，来查询接收缓冲区中剩余数据的大小，而无需直接读取数据。这使得应用程序能够更有效地决定何时进行数据读取，从而避免不必要的系统调用，减少延迟和提高性能。

使用场景：它主要用于需要精确控制数据读取时机的高性能网络应用程序，如 web 服务器、代理服务器、负载均衡器、数据库服务器等。