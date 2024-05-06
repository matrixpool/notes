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

- Question
    - what is synchronization vector of stream ciphers

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
