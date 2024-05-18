========================
TLS 
========================


- Four protocols 
    - the handshake protocol
    - the alert protocol
    - the change cipher spec protocol
    - the application data protocol.

- Connection states 
    - read and write states 
    - pending read and write states 
    - empty state 

- Crypto
    - MAC is computed before encryption

- handshake protocol负责协商一个会话，会话内容包括：
    - session identifier: server产生的唯一不重复的字符串
    - peer certificate: 对端的x509证书
    - compression method: 在加密之前进行有压缩
    - cipher spec:
        - pseudorandom function(PRF)用于产生密钥素材
        - block data encryption algorithm
        - MAC algorithm
    - master secret: 48-byte secret shared在服务端和客户端之间
    - 是否可恢复: 标识session是否可以被一个新连接重复使用
- alert protocol: alert message将立即中断连接，对应的seesionId将立即无效

- TLSv1.2和TLSv1.3的区别
    - 1.2中application data加密数据中序列号没有纳入加密，1.3则纳入了加密（序列号8bytes）
    - 进行GCM加密，1.2的关联数据包括SN、type、version（0x0303）和length，v1.3则只有type、version(0x0303)和length

- Question
    - what is synchronization vector of stream ciphers

TLS 1.3 为了与旧版本的 TLS 和加密软件保持兼容性，特意在记录层（Record Layer）保留了 TLS 1.2 的版本号（0x0303）。这一决策主要是出于以下原因：

- 中间盒兼容性：许多网络中间盒（如防火墙、入侵检测系统等）被配置为只允许特定版本的 TLS 流量通过。如果 TLS 1.3 在记录层使用了一个不同的版本号，这些中间盒可能无法正确处理 TLS 1.3 的流量，导致连接失败或数据丢失。

- 简化部署：通过使用与 TLS 1.2 相同的版本号，TLS 1.3 可以更容易地在现有的网络基础设施中部署，而不需要对中间盒进行升级或重新配置。

- 客户端和服务器的兼容性：一些客户端和服务器可能还没有升级到支持 TLS 1.3。通过在记录层使用 TLS 1.2 的版本号，TLS 1.3 可以与这些旧版本的客户端和服务器进行交互，同时提供更好的安全性。

过渡期的兼容性：在 TLS 1.3 逐渐被广泛采用的过渡期间，保持与 TLS 1.2 的兼容性可以减少部署和配置新协议的障碍。

KTLS AND OPENSSL
===============================

- client
    - SSL_set_fd ktls_enable
    - SSL_connect ktls_start
    - SSL_shutdown ktls_send_ctrl_message
- server
    - SSL_set_fd ktls_enable
    - SSL_accept ktls_start ktls_send_ctrl_message ktls_send_ctrl_message
    - SSL_shutdown ktls_send_ctrl_message
- performance
    - %2 data copy and 10% data crypto



