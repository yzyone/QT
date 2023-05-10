# QT使用OpenSSL的接口实现RSA的加密解密

2023-05-04 20:03·[C加加Qt技术开发老杰](https://www.toutiao.com/c/user/token/MS4wLjABAAAA_VcqaTCEm_1_AalUMfgVA5YbfLXBLC8yQm-9uPFMP2RqedFy_QegcjJdrcmdLZTK/?source=tuwen_detail)

![img](./images/8c42aa821bc04f4e8c5f4b33a27cf398.image)



首先介绍下命令台下openssl工具的简单使用：

- 生成一个密钥：

openssl genrsa -out test.key 1024

这里-out指定生成文件的。需要注意的是这个文件包含了公钥和密钥两部分，也就是说这个文件即可用来加密也可以用来解密。后面的1024是生成密钥的长度。

- openssl可以将这个文件中的公钥提取出来：

openssl rsa -in test.key -pubout -out test_pub.key

-in指定输入文件，-out指定提取生成公钥的文件名。至此，我们手上就有了一个公钥，一个私钥（包含公钥）。现在可以将用公钥来加密文件了。

- 我在目录中创建一个hello的文本文件，然后利用此前生成的公钥加密文件：

openssl rsautl -encrypt -in hello -inkey test_pub.key -pubin -out hello.en

-in指定要加密的文件，-inkey指定密钥，-pubin表明是用纯公钥文件加密，-out为加密后的文件。

- 解密文件：

openssl rsautl -decrypt -in hello.en -inkey test.key -out hello.de

-in指定被加密的文件，-inkey指定私钥文件，-out为解密后的文件。

```
OpenSSL> genrsa -out rsa_private_key.pem 1024
Generating RSA private key, 1024 bit long modulus
....+++++
.................+++++
e is 65537 (0x10001)
OpenSSL> genrsa -out rsa_private_key.pem 1024
Generating RSA private key, 1024 bit long modulus
..............+++++
..........................................+++++
e is 65537 (0x10001)
OpenSSL> rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
writing RSA key
OpenSSL> pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -out pkcs8_rsa_private_key.pem -nocrypt
OpenSSL> rsautl -encrypt -in hello.txt -inkey rsa_public_key.pem -pubin -out hello.en
OpenSSL> rsautl -decrypt -in hello.en -inkey rsa_private_key.pem -out hello.de
```

QT代码实现

test1.pro

```
QT += core
QT -= gui

CONFIG += c++11

TARGET = test1
CONFIG += console
CONFIG -= app_bundle

TEMPLATE = app

SOURCES += main.cpp \
#    rsasignature.cpp \
    rsa.cpp

LIBS += ../windows/x86/bin/libeay32.dll
LIBS += ../windows/x86/bin/ssleay32.dll
INCLUDEPATH += $$quote(../windows/x86/include/)

HEADERS += \
#    rsasignature.h \
    rsa.h

DESTDIR = $$PWD/
```

main.cpp

```
#include <QCoreApplication>
#include <QFile>
#include <QDebug>
#include "rsa.h"

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    /*公钥加密，私钥解密 测试案例*/
    QString strPlainData_before = QString::fromLocal8Bit("12dsadas云123345612dsadas12dsadas云123345612dsadas12dsadas云123345612dsadas12dsadas云123345612dsadas12dsadas云213123");
    QString strPubKey = "";
    QString strPriKey = "";
    QString strEncryptData = "";
    QString strPlainData = "";

    rsa *m_rsa = new rsa();

    QFile file("./rsa_public_key.pem");
    file.open(QIODevice::ReadOnly);
    strPubKey = file.readAll();

    QFile file1("./rsa_private_key.pem");
    file1.open(QIODevice::ReadOnly);
    strPriKey = file1.readAll();

    strEncryptData = m_rsa->rsaPubEncrypt(strPlainData_before,strPubKey);
    qDebug() << ("strEncryptData:"+strEncryptData);

    strPlainData = m_rsa->rsaPriDecrypt(strEncryptData, strPriKey);
    qDebug() << ("strPlainData:"+strPlainData);

    file.close();
    file1.close();


    return a.exec();
}
```

rsa.h

```
#ifndef RSA_H
#define RSA_H

#include <QString>
#include <openssl/rsa.h>
#include <openssl/pem.h>
#include <openssl/err.h>

#define BEGIN_RSA_PUBLIC_KEY    "BEGIN RSA PUBLIC KEY"
#define BEGIN_RSA_PRIVATE_KEY   "BEGIN RSA PRIVATE KEY"
#define BEGIN_PUBLIC_KEY        "BEGIN PUBLIC KEY"
#define BEGIN_PRIVATE_KEY       "BEGIN PRIVATE KEY"
#define KEY_LENGTH              1024

class rsa
{
public:
    rsa();
    ~rsa();

    QString rsaPubEncrypt(const QString &strPlainData, const QString &strPubKey);

    QString rsaPriDecrypt(const QString &strDecryptData, const QString &strPriKey);
};

#endif // RSA_H
```

rsa.cpp

```
#include "rsa.h"

rsa::rsa()
{

}

rsa::~rsa()
{

}

QString rsa::rsaPubEncrypt(const QString &strPlainData, const QString &strPubKey)
{
    QByteArray pubKeyArry = strPubKey.toUtf8();
    uchar* pPubKey = (uchar*)pubKeyArry.data();
    BIO* pKeyBio = BIO_new_mem_buf(pPubKey, pubKeyArry.length());
    if (pKeyBio == NULL) {
        return "";
    }


    RSA* pRsa = RSA_new();
    if (strPubKey.contains(BEGIN_RSA_PUBLIC_KEY)) {
        pRsa = PEM_read_bio_RSAPublicKey(pKeyBio, &pRsa, NULL, NULL);

    }
    else {
        pRsa = PEM_read_bio_RSA_PUBKEY(pKeyBio, &pRsa, NULL, NULL);

    }


    if (pRsa == NULL) {
        BIO_free_all(pKeyBio);
        return "";
    }

    int nLen = RSA_size(pRsa);
    char* pEncryptBuf = new char[nLen];


    //加密
    QByteArray plainDataArry = strPlainData.toUtf8();
    int nPlainDataLen = plainDataArry.length();

    int exppadding=nLen;
    if(nPlainDataLen>exppadding-11)
        exppadding=exppadding-11;
    int slice=nPlainDataLen/exppadding;//片数
    if(nPlainDataLen%(exppadding))
        slice++;

    QString strEncryptData = "";
    QByteArray arry;
    for(int i=0; i<slice; i++)
    {
        QByteArray baData = plainDataArry.mid(i*exppadding, exppadding);
        nPlainDataLen = baData.length();
        memset(pEncryptBuf, 0, nLen);
        uchar* pPlainData = (uchar*)baData.data();
        int nSize = RSA_public_encrypt(nPlainDataLen,
                                       pPlainData,
                                       (uchar*)pEncryptBuf,
                                       pRsa,
                                       RSA_PKCS1_PADDING);
        if (nSize >= 0)
        {
            arry.append(QByteArray(pEncryptBuf, nSize));
        }
    }

    strEncryptData += arry.toBase64();
    //释放内存
    delete pEncryptBuf;
    BIO_free_all(pKeyBio);
    RSA_free(pRsa);

    return strEncryptData;
}

QString rsa::rsaPriDecrypt(const QString &strDecryptData, const QString &strPriKey)
{
    QByteArray priKeyArry = strPriKey.toUtf8();
    uchar* pPriKey = (uchar*)priKeyArry.data();
    BIO* pKeyBio = BIO_new_mem_buf(pPriKey, priKeyArry.length());
    if (pKeyBio == NULL) {
        return "";
    }

    RSA* pRsa = RSA_new();
    pRsa = PEM_read_bio_RSAPrivateKey(pKeyBio, &pRsa, NULL, NULL);
    if (pRsa == NULL) {
        BIO_free_all(pKeyBio);
        return "";
    }

    int nLen = RSA_size(pRsa);
    char* pPlainBuf = new char[nLen];

    //解密
    QByteArray decryptDataArry = strDecryptData.toUtf8();
    decryptDataArry = QByteArray::fromBase64(decryptDataArry);
    int nDecryptDataLen = decryptDataArry.length();

    int rsasize=nLen;
    int slice=nDecryptDataLen/rsasize;//片数
    if(nDecryptDataLen%(rsasize))
        slice++;

    QString strPlainData = "";
    for(int i=0; i<slice; i++)
    {
        QByteArray baData = decryptDataArry.mid(i*rsasize, rsasize);
        nDecryptDataLen = baData.length();
        memset(pPlainBuf, 0, nLen);
        uchar* pDecryptData = (uchar*)baData.data();
        int nSize = RSA_private_decrypt(nDecryptDataLen,
                                        pDecryptData,
                                        (uchar*)pPlainBuf,
                                        pRsa,
                                        RSA_PKCS1_PADDING);
        if (nSize >= 0) {
            strPlainData += QByteArray(pPlainBuf, nSize);
        }
    }

    //释放内存
    delete pPlainBuf;
    BIO_free_all(pKeyBio);
    RSA_free(pRsa);

    return strPlainData;
}
```



------

**原文链接：[「链接」](http://qt.0voice.com/?id=1118)**

**资料领取：[Qt资料领取（视频教程+文档+代码+项目实战）](http://docs.qq.com/doc/DRkFpUUJGWEtjemhX)**