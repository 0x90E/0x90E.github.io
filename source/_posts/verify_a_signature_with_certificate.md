---
title: Verify a signature with certificate
date: 2017-02-12 15:24:52
tags: Openssl
---

### 所需檔案建立
#### 建立CA基礎設施
1. 建立CA基礎設施，步驟可參考[openssl-certificate-authority](https://jamielinux.com/docs/openssl-certificate-authority/introduction.html)
2. 透過以上步驟可獲得Certificate與Private key
    - Certificate位置: /root/ca/intermediate/certs/www.example.com.cert.pem
    - Private key位置: /root/ca/intermediate/private/www.example.com.key.pem

<br>

#### 建立Signature
1. 可以使用前述所產生的Certificate與Private key，或已經擁有的Certificate與Private key產生Signature，只要Certificate與Private key兩者是有對應關西的。
2. ``` bash    
    $ echo "hello signature" > test.txt  
    $ openssl dgst -sha256 -sign /root/ca/intermediate/private/www.example.com.key.pem -out test.txt.sha256 test.txt
    ```
<br>

#### 使用Certificate驗證Signature
1. Certificate無法直接驗證Signature，需先透過指令從Certificate提取出Public key，再用Pubilc key驗證Signature是否合法
    ``` bash
    $ openssl x509 -pubkey -noout -in /root/ca/intermediate/certs/www.example.com.cert.pem > www.example.com.pubkey.pem
    $ openssl dgst -verify www.example.com.pubkey.pem -signature test.txt.sha256 test.txt
    Verified OK
    ```

<br>

### 使用C++實做

#### C++ example code
``` c++
/* ------------------------------------------------------------ *
 * file:        verifySignature.cpp                             *
 * purpose:     Example code for OpenSSL signature validation   *
 *                                                              *
 * g++ -o verifySignature verifySignature.cpp -lssl -lcrypto    *
 * ------------------------------------------------------------ */

#include <stdio.h>
#include <stdlib.h>
#include <string>
#include <openssl/bio.h>
#include <openssl/err.h>
#include <openssl/pem.h>
#include <openssl/x509.h>
#include <openssl/x509_vfy.h>
#include <openssl/crypto.h>

bool verifySignature(std::string certFile, std::string sigFile, std::string sourceFile) {
    BIO *certFileBio = NULL, *sigFileBio = NULL, *sourceFileBio = NULL, 
        *digestBio = NULL, *messageDigestBio = NULL, *bio_err = NULL;   
    EVP_PKEY *sigPublickey = NULL;
    X509 *cert = NULL;
    EVP_MD_CTX *mdCtx = NULL;
    int i = 0;
    size_t sigLen = 0;
    unsigned char *sigBuf = NULL, *digestBuf = NULL;
    bool ret = false, readFileFailed = false;

    do {
        // Create the Input/Output BIO's.
        bio_err = BIO_new_fp(stdout, BIO_NOCLOSE);
        if (!bio_err) {
            BIO_printf(bio_err, "Fail to create the output BIO's \n");
            ERR_print_errors(bio_err);
            break;
        }
        certFileBio = BIO_new(BIO_s_file());
        if (!certFileBio) {
            BIO_printf(bio_err, "Fail to create the certificate file BIO's \n");
            ERR_print_errors(bio_err);
            break;
        }

        // Load the certification from file (PEM).
        if (!BIO_read_filename(certFileBio, certFile.c_str())) {
            break;
        }
        if (!(cert = PEM_read_bio_X509(certFileBio, NULL, 0, NULL))) {
            BIO_printf(bio_err, "Error loading cert into memory \n");
            ERR_print_errors(bio_err);
            break;
        }

        // Load public key from certification.
        sigPublickey = X509_get_pubkey(cert);
        if (!sigPublickey) {
            BIO_printf(bio_err, "Fail to get public key form certificate \n");
            ERR_print_errors(bio_err);
            break;
        }

        // Create the message digest BIO.
        messageDigestBio = BIO_new(BIO_f_md());
        if (!messageDigestBio) {
            IO_printf(bio_err, "Fail to create the message digest BIO's \n");
            ERR_print_errors(bio_err);
            break;
        }

        // Get digest BIOs context.
        if (!BIO_get_md_ctx(messageDigestBio, &mdCtx)) {
            BIO_printf(bio_err, "Error getting context\n");
            ERR_print_errors(bio_err);
            break;
        }

        // Sets up verification context to use digest.
        if (!EVP_DigestVerifyInit(mdCtx, NULL, EVP_sha256(), NULL, sigPublickey)) {
            BIO_printf(bio_err, "Error setting context\n");
            ERR_print_errors(bio_err);
            break;
        }
        
        // Read signature file into buffer.
        sigFileBio = BIO_new_file(sigFile.c_str(), "rb");
        if (!sigFileBio) {
            BIO_printf(bio_err, "Error opening signature file %s\n", sigFile);
            ERR_print_errors(bio_err);
            break;
        }
        sigLen = EVP_PKEY_size(sigPublickey);
        sigBuf = (unsigned char*) OPENSSL_malloc(sigLen);
        if (sigBuf == NULL) {
            BIO_printf(bio_err, "Could not allocate %d bytes for %s\n",
                    sigLen, "signature buffer");
            ERR_print_errors(bio_err);
            break;
        }
        sigLen = BIO_read(sigFileBio, sigBuf, sigLen);
        if (sigLen <= 0) {
            BIO_printf(bio_err, "Error reading signature file %s\n", sigFile);
            ERR_print_errors(bio_err);
            break;
        }      

        // Append the source file to digest BIO.
        sourceFileBio = BIO_new(BIO_s_file());
        if (!sourceFileBio) {
            break;
        }
        if (!BIO_read_filename(sourceFileBio, sourceFile.c_str())) {
            break;
        }
        digestBio = BIO_push(messageDigestBio, sourceFileBio);

        // Move data pointer position to the end.
        digestBuf = (unsigned char*) OPENSSL_malloc(BUFSIZE);
        if (digestBuf == NULL) {
            BIO_printf(bio_err, "Could not allocate %d bytes for %s\n",
                    BUFSIZE, "I/O buffer");
            ERR_print_errors(bio_err);
            break;
        }
        for (;;) {
            i = BIO_read(digestBio, (char *)digestBuf, BUFSIZE);
            if (i < 0) {cert/VortexModel_v3.bin.sha256
                BIO_printf(bio_err, "Read Error\n");
                ERR_print_errors(bio_err);
                readFileFailed = true;
            }
            if (i == 0)
                break;
        }
        if (readFileFailed) {
            break;
        }
        
        // Load the digest BIOs context into EVP_MD_CTX
        BIO_get_md_ctx(digestBio, &mdCtx);
        i = EVP_DigestVerifyFinal(mdCtx, sigBuf, sigLen);
        if (i > 0) {
            BIO_printf(bio_err, "Verified OK\n");
            ret = true;
        }
        else if (i == 0) {
            BIO_printf(bio_err, "Verification Failure\n");
        } else {
            BIO_printf(bio_err, "Error Verifying Data\n");
            ERR_print_errors(bio_err);
        }        
    } while(0);

    if (bio_err) {
        BIO_free_all(bio_err);
    }
    if (certFileBio) {
        BIO_free_all(certFileBio);
    }
    if (cert) {
        X509_free(cert);
    }    
    if (sigPublickey) {
        EVP_PKEY_free(sigPublickey);
    }
    if (messageDigestBio) {
        BIO_free(messageDigestBio);
    }
    if (sigFileBio) {
        BIO_free(sigFileBio);
    }
    if (sigBuf) {
        OPENSSL_free(sigBuf);
    }
    if (digestBuf) {
        OPENSSL_free(digestBuf);
    }    
    if (sourceFileBio) {
        BIO_free(sourceFileBio);
    }
    return ret;
}

int main(int argc, char** argv) {
    std::string certFile("/root/ca/intermediate/certs/www.example.com.cert.pem");
    std::string sigFile("test.txt.sha256");
    std::string sourceFile("test.txt");
    
    verifySignature(certFile, sigFile, sourceFile);

    return 0;    
}
```

### Compiling the Code
``` bash
$ gcc -o verifySignature verifySignature.c -lssl -lcrypto
$ ./verifySignature
```
