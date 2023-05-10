# Qt实践-HTTPS知识点-SSL socket获取百度首页

2023-04-26 17:46·[QT教程](https://www.toutiao.com/c/user/token/MS4wLjABAAAAJW6PdNNpohsjPXV2WiU3PbO3EgXSURUlbocojcbC-p_r9xf5l28xylvylyEMnUST/?source=tuwen_detail)

**基本概念**

这里要明确一点，HTTP/HTTPS是应用层协议，而socket一般指TCP/UDP协议，也就是在传输层中，而IP协议是在网络层中！

这个实例主要是撸socket，然后手动构造HTTP包，完成应用层的功能。

这里使用了C++中的Qt框架



**代码与实例**

程序运行截图如下：



![img](./images/f3da3ffdaa7944748369e94a0c95f0a6.image)





源码如下：

```
#include <QCoreApplication>
#include <QSslSocket>
#include <QObject>
#include <QDebug>
#include <QEventLoop>


int main(int argc, char *argv[])
{
QCoreApplication a(argc, argv);

QSslSocket *socket = new QSslSocket;
socket->connectToHostEncrypted("www.baidu.com", 443);

QEventLoop loop;
QObject::connect(socket, SIGNAL(encrypted()), &loop, SLOT(quit()));
loop.exec();

QObject::connect(socket, &QSslSocket::stateChanged, [=](){

qDebug() << "ssl socket断了，这个网站有毒！快跑！";
});

socket->write("GET https://www.baidu.com/ HTTP/1.1\r\n"
"Host: www.baidu.com\r\n"
"User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:49.0) Gecko/20100101 Firefox/49.0\r\n"
"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
"Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3\r\n"
"Accept-Encoding: deflate, br\r\n"
"Cookie: BAIDUID=AFAC3106555A50E243924D01EB575871:FG=1; BIDUPSID=0770F8E70DCC708B09D0A4EB13FA4EA9; PSTM=1566473759; BD_UPN=13314352; COOKIE_SESSION=7_1_5_1_2_3_0_1_4_1_0_3_2196_0_89_0_1566912600_1566910408_1566912511%7C9%231980_13_1566912293%7C6; delPer=0; BD_HOME=0; H_PS_PSSID=1461_21120_29523_29518_29099_29567_29221_29072\r\n"
"Connection: keep-alive\r\n"
"Upgrade-Insecure-Requests: 1\r\n"
"\r\n");

QObject::connect(socket, SIGNAL(readyRead()), &loop, SLOT(quit()));
loop.exec();

qDebug() << QString::fromUtf8(socket->readAll());

delete socket;

qDebug() << "over!";

return a.exec();
}
```

**【领QT开发教程学习资料，点击下方链接莬费领取↓↓，先码住不迷路~】**

**点击这里：[「链接」](http://docs.qq.com/doc/DUlVwTW1FZlZuWE9G)**