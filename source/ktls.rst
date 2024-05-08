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