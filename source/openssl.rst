
Tongsuo
===================

SSL连接：rbio和wbio复用同一个bio，bio的method为bss_sock.c中的实现

SSL client WRTITE
----------------------
main->SSL_connect->SSL_do_handshake->ossl_statem_connect->state_machine->write_state_machine
->ossl_statem_client_post_work->tls1_change_cipher_state->BIO_ctrl->ktls_start(TX)

SSL client READ
----------------------
main->SSL_connect->SSL_do_handshake->ossl_statem_connect->state_machine->read_state_machine
->ossl_statem_client_process_message->tls_process_change_cipher_spec->ssl3_do_change_cipher_spec
->tls1_change_cipher_state->BIO_ctrl->ktls_start(RX)
