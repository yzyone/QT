# Qt怎么使用MD5加密

2023-05-04 19:47·[QT高级进阶](https://www.toutiao.com/c/user/token/MS4wLjABAAAAt_BOijoaoSs32bWWP-4kn4w4vK6Ml7vxKe16MMXkZiuF3N4YH95xKOtz9OB6zOVG/?source=tuwen_detail)

Qt中包含了大部分常用的功能，比如json、数据库、网络通信、串口通信以及今天说的这个MD5加密；

Qt不只是一个图像界面库，更是一个C++的类库和框架。在嵌入式linux系统开发时，可以将qt移植过去，然后基于qt的框架进行应用开发，可以大大提高开发效率，C++里面封装的类库与数据类型，使用起来很方便，都有点像java了。

在开发时，有时会用到MD5加密去保存密码，因此需要用到MD5算法，网络上也可以找到纯C实现的代码，虽然不太复杂，但是也需要花点时间去找和测试。qt里面已经封装了MD5的加密算法，并且使用起来很是方便。

Qt中将字符串进行MD5加密其实比较简单，代码如下：

```
#include <QCoreApplication>
#include <QCryptographicHash>
#include <QDebug>
 
int main(int argc, char *argv[])
{
 QCoreApplication a(argc, argv);
 
 
 QString str = "123456";
 QString md5Str = QCryptographicHash::hash(str.toLatin1(),QCryptographicHash::Md5).toHex();
 
 qDebug()<<"md5: "<<md5Str;
 
 return a.exec();
}
```

有这么简单的接口，还需要找C代码吗？

执行结果：

```
md5: "e10adc3949ba59abbe56e057f20f883e"
```

MD5加密是不可逆的（不过现在据说有破解的），我们在程序中如果是使用MD5加密去保存密码的话，那么对比密码时，需要转换为MD5字符串后进行对比。