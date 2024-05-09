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
                                                    |--->tls_set_sw_offload->crypto_alloc_aead
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
