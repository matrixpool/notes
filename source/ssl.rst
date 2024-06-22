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

GCM模式
============================

GCM输入参数:
- IV (12Bytes)
    由4字节write IV和8字节nonce组成，write IV有密钥派生RPF产生（）。nonce是随机产生，使用时先加1。
    见https://datatracker.ietf.org/doc/html/rfc5246#section-6.2.3.3

- aad (13Bytes)
    由8字节rec_seq(初始必须为0，使用前加1)，1字节记录类型、2字节版本和2字节长度组成。

下面是使用openssl对TLS GCM-AES256加密数据的验证代码:

.. code-block:: c

    #include <openssl/evp.h>
    #include <openssl/rand.h>
    #include <stdio.h>
    #include <string.h>

    int aes_gcm_encrypt(const unsigned char *plaintext, int plaintext_len,
                        const unsigned char *aad, int aad_len,
                        const unsigned char *key, const unsigned char *iv, int iv_len,
                        unsigned char *ciphertext, unsigned char *tag) {
        EVP_CIPHER_CTX *ctx;
        int len;
        int ciphertext_len;

        // 创建并初始化上下文
        if(!(ctx = EVP_CIPHER_CTX_new())) return -1;

        // 初始化加密操作
        if(1 != EVP_CipherInit_ex(ctx, EVP_aes_256_gcm(), NULL, NULL, NULL, 1)) return -1;

        // 设置IV长度，如果不同于默认的12字节
        if(1 != EVP_CIPHER_CTX_ctrl(ctx, EVP_CTRL_GCM_SET_IVLEN, iv_len, NULL)) return -1;

        // 初始化密钥和IV
        if(1 != EVP_CipherInit_ex(ctx, NULL, NULL, key, iv, 1)) return -1;

        // 提供AAD数据
        if(aad && aad_len > 0) {
            if(1 != EVP_CipherUpdate(ctx, NULL, &len, aad, aad_len)) return -1;
        }

        // 提供要加密的消息，并得到加密后的输出
        if(1 != EVP_CipherUpdate(ctx, ciphertext, &len, plaintext, plaintext_len)) return -1;
        ciphertext_len = len;

        // 完成加密操作
        if(1 != EVP_CipherFinal_ex(ctx, ciphertext + len, &len)) return -1;
        ciphertext_len += len;

        // 获取认证标签
        if(1 != EVP_CIPHER_CTX_ctrl(ctx, EVP_CTRL_GCM_GET_TAG, 16, tag)) return -1;

        // 清理
        EVP_CIPHER_CTX_free(ctx);

        return ciphertext_len;
    }

    int main() {
        unsigned char tag[16] = {0};
        unsigned char plaintext[13] = {
            0x61, 0x61, 0x61, 0x61, 0x61, 0x61, 0x61, 0x61, 
            0x61, 0x61, 0x61, 0x61, 0x61
        };
        
        // 密钥和 IV
        unsigned char key[] = {
            0x30, 0x90, 0x55, 0xba, 0xc0, 0x80, 0x12, 0xbb, 
            0x12, 0x00, 0xc4, 0xfc, 0x14, 0xd3, 0x08, 0x40, 
            0xac, 0xce, 0x90, 0xd3, 0x69, 0x99, 0x99, 0xf2, 
            0xcd, 0x30, 0x16, 0x1f, 0x26, 0xc9, 0x2d, 0x5d, 
        };
        unsigned char iv[] = {
            0xac, 0x8d, 0xe0, 0xfd, 0xf6, 0x9a, 0x03, 0x06, 0x69, 0x3a, 0xfc, 0xc2, 
        };

        unsigned char aad[] = {
            0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
            0x17, 0x03, 0x03, 0x00, 0x25
        };

        // 缓冲区用于存储密文
        unsigned char ciphertext[sizeof(plaintext)];

        // 执行加密
        int ciphertext_len = aes_gcm_encrypt(plaintext, sizeof(plaintext), aad, sizeof(aad), key, iv, sizeof(iv), ciphertext, tag);

        if (ciphertext_len < 0) {
            fprintf(stderr, "Encryption failed\n");
            return 1;
        }

        // 输出加密结果
        printf("Ciphertext is:\n");
        for (int i = 0; i < ciphertext_len; i++) {
            printf("%02x", ciphertext[i]);
        }
        printf("\n");

        printf("Tag is:\n");
        for (int i = 0; i < sizeof(tag); i++) {
            printf("%02x", tag[i]);
        }
        printf("\n");

        return 0;
    }






