# Qt多线程实现网络发送文件功能

客户端给服务器发送文件，服务器进行接收文件的简单操作

# 1. 服务器

1. 创建QTcpServer 类的对象

```
QTcpServer * server = new QTcpServer(this);
```

2. 进行监听

```
bool QTcpServer::listen(const QHostAddress &address = QHostAddress::Any, quint16 port = 0)
```

3. 通过接收 QTcpServer 发出的 newConnection 的信号，进行下一步操作

```
[signal] void QTcpServer::newConnection()
```

4. 通过调用 nextPendingConnection 方法获取套接字

```
// 通过 this->m_server 调用 nextPendConnection
QTcpSocket * socket = server->nextPendingConnection();
```

5. 接收客户端发来是消息 通过 [signal] void QIODevice::readyRead() 信号

6.客户端下线 [signal] void
QAbstractSocket::disconnected() 信号 表示

创建一个子线程类，继承 QThread ,重写父类的run() 方法

在run方法中，创建文件，接收客户端发的文件写进创建的文件中；

接收文件时，要先获取第一次客户端发来的文件大小；

获取客户端第一次发来的文件大小

```
// 进行接收数据的时候，需要知道客户端发来的文件的大小
// 先将客户端第一次发来的数据的大小读取出来
static int count = 0;   // 判断是否是客户端第一次发来的数据
static int total = 0;   // 记录文件的大小
        if(count == 0)
        {
            this->m_tcp->read((char*)&total, 4);    // 获取文件大小
        }
```

创建子线程类 并启动子线程

```
// 创建子线程类对象
MyQThread * myqtread = new MyQThread;
// 启动子线程
myqtread->start();
```

服务端代码：

widget.h 主线程头文件

```
#ifndef WIDGET_H
#define WIDGET_H
 
#include <QWidget>
#include <QTcpServer>
 
namespace Ui {
class Widget;
}
 
class Widget : public QWidget
{
    Q_OBJECT
public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();
private slots:
    void on_listenBtn_clicked();
private:
    // 创建QTcpServer 类的对象
    QTcpServer * m_server;
private:
    Ui::Widget *ui;
};
 
#endif // WIDGET_H
QT开发交流+赀料君羊：661714027
```

widget.cpp 主线程：

```
#include "widget.h"
#include "ui_widget.h"
 
#include "myqthread.h"
#include <QMessageBox>
 
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
 
    // 设置端口号
    ui->port->setText("8989");
    // 利用多线程进行链接服务器
    // 1. 需要创建一个线程类的子类 ，让其继承Qt中的线程QThread
    // 2. 重写父类的run() 方法，在该函数内部编写子线程要处理的具体业务流程
    // 3. 在主线程中创建子线程对象，new 一个就可以
    // 4. 启动子线程，调用start() 方法
    // 实例化QTcpServer 对象
    this->m_server = new QTcpServer(this);
    // 检验是否接收客户端的连接
    connect(this->m_server, &QTcpServer::newConnection, this, [=]()
    {
        // 获取套接字
        QTcpSocket * tcp = this->m_server->nextPendingConnection();
        // 创建子线程类对象
        MyQThread * myqtread = new MyQThread(tcp);
        // 启动子线程
        myqtread->start();
        // 获取子线程中发来的客户端端口的消息
        connect(myqtread, &MyQThread::ClientDisconnect, this, [=]()
        {
            //弹出对话框提示
            QMessageBox::warning(this, "警告", "客户端已断开连接...");
        });
        // 接收接收完客户端的信号
        connect(myqtread, &MyQThread::OverRecveid, this, [=]()
        {
            //弹出对话框提示
            QMessageBox::information(this, "提示", "已接收文客户端发来的数据");
            // 关闭套接字
            tcp->close();
            // 释放
            tcp->deleteLater();
            // 释放线程
            myqtread->quit();
            myqtread->wait();
            myqtread->deleteLater();
        });
    });
}
 
Widget::~Widget()
{
    delete ui;
}
// 点击监听按钮 进行监听 按钮转到槽的方式
void Widget::on_listenBtn_clicked()
{
    //获取端口号
    unsigned short port = ui->port->text().toUShort();
    //利用this->m_s 调用listen 进行监听
    this->m_server->listen(QHostAddress::Any, port);
}
```

myqthread.h 子线程头文件

```
#ifndef MYQTHREAD_H
#define MYQTHREAD_H
 
//#include <QObject>
 
#include <QTcpSocket>
#include <QThread>
 
class MyQThread : public QThread
{
    Q_OBJECT
public:
    explicit MyQThread(QTcpSocket *tcp, QObject *parent = nullptr);
 
    // 2.重写QThread 类中的受保护成员 run() 方法
protected:
    void run();
 
public:
    // 自定义套接字对象 记录主线程传进的套接字对象 tcp
    QTcpSocket * m_tcp;
 
signals:
    // 自定义信号 将服务器接收完客户端发来的数据 告诉主线程
    void OverRecveid();
    // 自定义信号 将客户端断开连接 告诉主线程
    void ClientDisconnect();
 
public slots:
};
 
#endif // MYQTHREAD_H
```

myqthread.cpp 子线程文件

```
#include "myqthread.h"
 
#include <QFile>
 
MyQThread::MyQThread(QTcpSocket *tcp, QObject *parent) : QThread(parent)
{
    this->m_tcp = tcp;
}
// 2.重写QThread 类中的受保护成员 run() 方法
void MyQThread::run()
{
    // 1.创建文件 打开文件
    QFile * file = new QFile("recv.txt");
    file->open(QFile::WriteOnly);   // 以只写的方式打开文件
    // 2.检验是否进行读写
    connect(this->m_tcp, &QTcpSocket::readyRead, this, [=]()
    {
        // 进行接收数据的时候，需要知道客户端发来的文件的大小
        // 先将客户端第一次发来的数据的大小读取出来
        static int count = 0;   // 判断是否是客户端第一次发来的数据
        static int total = 0;   // 记录文件的大小
        if(count == 0)
        {
            this->m_tcp->read((char*)&total, 4);    // 获取文件大小
        }
        // 将剩下的数据全部读取出来
        // 获取客户端发来的数据
        QByteArray recvClient = this->m_tcp->readAll(); // 全部接收
        // 将读取的数据的量记录到count中
        count += recvClient.size();
        // 将数据写进文件中
        file->write(recvClient);
        // 判断服务器是否把客户端发来的数据全部读取完
        if(count == total)
        {
            // 关闭套接字
            this->m_tcp->close();
            // 释放套接字
            this->m_tcp->deleteLater();
            // 关闭文件
            file->close();
            file->deleteLater();
            // 自定义一个信号 告诉主线程文件 已接收完毕
            emit OverRecveid();
        }
    });
    // 3.检验客户端是否断开连接
    connect(m_tcp, &QTcpSocket::disconnected, this, [=]()
    {
        // 将客户端断开连接 发送给主线程
        emit this->ClientDisconnect();
    });
    // 调用 exec 进入事件循环 阻塞
    exec();
}
```

![img](E:\GitHub\QT\run\9579e8e4313b403cb4eb309df40d38fd~noop.image)



# 2.客户端

1. 绑定 ip 和 端口号

```
[virtual] void QAbstractSocket::connectToHost(const QString &hostName, quint16 port, OpenMode openMode = ReadWrite, NetworkLayerProtocol protocol = AnyIPProtocol)
```

2. 连接服务器

```
[signal] void QAbstractSocket::connected()
```

3. 通过套接字 调用 write方法发送消息给服务器

```
qint64 QIODevice::write(const char *data, qint64 maxSize)
```

4. 断开连接

```
[signal] void QAbstractSocket::disconnected()
```

利用多线程实现 选择文件 发送文件

利用第二种多线程的方法

1.创建一个新的类，让这个类从QObject中派生
2.在这个新的类中添加一个公有的成员函数，函数体是我们要子线程中执行的业务逻辑
3.在主线程中创建一个QThread对象，这个就是子线程的对象
4.在主线程中创建一个工作类的对象
5.将工作类对象移动到子线程对象中，需要调用QObject类提供的moveThread
6.启动子线程，调用start() 这个线程启动了，当时移动到线程中的对象并没有工作
7.调用工作类的对象函数，让这个函数开始执行，这个时候是在移动到那个子线程中运行的。

客户端代码：

mythread.h 任务类头文件

```
#ifndef MYTHREAD_H
#define MYTHREAD_H
 
#include <QObject>
#include <QTcpSocket>
 
class MyThread : public QObject
{
    Q_OBJECT
public:
    explicit MyThread(QObject *parent = nullptr);
 
    // 连接服务器
    void connectToServer(unsigned short port, QString ip);
    // 发送文件
    void SendFile(QString path);
 
private:
    // 创建QTcpSocket 类的对象
    QTcpSocket * m_socket;
 
signals:
    // 自定义一个信息 告诉主线程 成功连接到服务器
    void ConnectOK();
 
    // 自定义一个信号 告诉主线程服务器已断开连接
    void gameOver();
 
    // 自定义一个信号 将获取的百分比发送给主线程
    void SendPercent(int);
 
public slots:
};
 
#endif // MYTHREAD_H
```

mythread.cpp 任务类文件

```
#include "mythread.h"
 
#include <QFileInfo>
#include <QMessageBox>
 
MyThread::MyThread(QObject *parent) : QObject(parent)
{
 
}
// 连接服务器
void MyThread::connectToServer(unsigned short port, QString ip)
{
    // 实例化socket类的对象
    this->m_socket = new QTcpSocket(this);
    // 尝试与服务器取得连接 绑定IP 和端口号
    this->m_socket->connectToHost(ip, port);
    // 检验是否成功与服务器取等连接
    connect(this->m_socket, &QTcpSocket::connected, this, [=]()
    {
        emit this->ConnectOK(); // 自定义一个信号 告诉主线程 成功连接上服务器
    });
    // 检验服务器是否断开连接
    connect(this->m_socket, &QTcpSocket::disconnected, this, [=]()
    {
        this->m_socket->close();    // 关闭套接字
        emit this->gameOver();      // 发送信号 告诉主线程 服务器已断开连接
    });
}
// 发送文件
void MyThread::SendFile(QString path)
{
    // 1.获取文件大小
    QFileInfo info(path);
    int fileSize = info.size();
    // 2.打开文件
    QFile file(path);
    bool ret = file.open(QFile::ReadOnly);
    if(!ret)
    {
        QMessageBox::warning(NULL, "警告", "打开文件失败");
        return; // 退出函数
    }
    // 判断什么时候读完文件
    while(!file.atEnd())
    {
        // 第一次发送文件的时候 将文件的大小发送给服务器
        // 定义一个标记 当标记为0时， 表示第一次发送文件
        static int num = 0;
        if(num == 0)
        {
            this->m_socket->write((char*)&fileSize, 4); // 将文件大小发送给服务器
        }
        // 在循环体中 每次读取一行
        QByteArray line = file.readLine();
        // 每次发送一次数据，就将发送的数据的量记录下来 用于更新进度条
        num += line.size();
        // 基于num值 计算百分比
        int percent = (num*100/fileSize);
        // 将百分比发送给主线程
        emit this->SendPercent(percent);
        // 将读取的数据通过套接字对象发送给服务器
        this->m_socket->write(line);
    }
}
```

widget.h 主线程头文件

```
#ifndef WIDGET_H
#define WIDGET_H
 
#include <QWidget>
 
namespace Ui {
class Widget;
}
 
class Widget : public QWidget
{
    Q_OBJECT
 
public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();
 
signals:
    // 自定义一个信号 告诉子线程进行链接服务器
    void TellToConnect(unsigned short port, QString ip);
    // 自定义一个信号 将选中的文件路径发送给任务类
    void SendToFile(QString);
 
private slots:
    void on_connectBtn_clicked();
 
    void on_selectBtn_clicked();
 
    void on_sendBtn_clicked();
 
private:
    QString m_path;
private:
    Ui::Widget *ui;
};
 
#endif // WIDGET_H
```

widget.cpp

```
#include "widget.h"
#include "ui_widget.h"
 
#include <QFileDialog>
#include <QMessageBox>
#include <QThread>
#include "mythread.h"
 
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
 
    // 利用多线程实现 选择文件 发送文件
    // 利用第二种多线程的方法
    // 1.创建一个新的类，让这个类从QObject中派生
    // 2.在这个新的类中添加一个公有的成员函数，函数体是我们要子线程中执行的业务逻辑
    // 3.在主线程中创建一个QThread对象，这个就是子线程的对象
    // 4.在主线程中创建一个工作类的对象
    // 5.将工作类对象移动到子线程对象中，需要调用QObject类提供的moveThread方法
    // 6.启动子线程，调用start() 这个线程启动了，当时移动到线程中的对象并没有工作
    // 7.调用工作类的对象函数，让这个函数开始执行，这个时候是在移动到那个子线程中运行的。
 
    // 1.创建QThread对象
    QThread *t = new QThread;
    // 2.创建任务类的对象
    MyThread * working = new MyThread;
    // 3.将任务类对象移动到子线程中
    working->moveToThread(t);
    // 启动子线程
    t->start();
    // 4.设置IP 端口号
    ui->ip_lineEide->setText("127.0.0.1");
    ui->port_lineEdit->setText("8989");
    // 5.设置进度条
    ui->progressBar->setRange(0, 100);  // 进度条的范围
    ui->progressBar->setValue(0);       // 初始化为0
    // 6.更新进度条 通过连接任务类发来的信号 实现
    connect(working, &MyThread::SendPercent, ui->progressBar, &QProgressBar::setValue);
    // 7.接收任务类发来的成功连接到服务器 信号
    connect(working, &MyThread::ConnectOK, this, [=]()
    {
        QMessageBox::information(this, "提示", "成功连接到服务器");
        // 将文件按钮设置成不可用状态
        ui->sendBtn->setDisabled(false);
    });
    // 8.连接任务类发来的断开连接的信号
    connect(working, &MyThread::gameOver, this, [=]()
    {
        QMessageBox::warning(this, "警告", "服务器已断开连接");
        //释放支援
        t->quit();
        t->wait();
        t->deleteLater();
        working->deleteLater();
        // 将文件按钮设置成可用状态
        ui->sendBtn->setDisabled(true);
    });
    // 7.将信号和工作类对象中的任务函数连接
    connect(this, &Widget::TellToConnect, working, &MyThread::connectToServer);
    // 8.将文件路径发给任务函数
    connect(this, &Widget::SendToFile, working, &MyThread::SendFile);
    // 9.将发送文件按钮设置成可用状态
    ui->sendBtn->setDisabled(true);
}
 
Widget::~Widget()
{
    delete ui;
}
// 连接服务器
void Widget::on_connectBtn_clicked()
{
    // 获取ip 和 端口号
    QString ip = ui->ip_lineEide->text();
    unsigned short port = ui->port_lineEdit->text().toShort();
    // 将ip 和 端口号 发送取出
    emit this->TellToConnect(port, ip);
    // 将发送文件按钮设置成不可用状态
    ui->sendBtn->setDisabled(false);
}
// 选中文件
void Widget::on_selectBtn_clicked()
{
    m_path = QFileDialog::getOpenFileName();  // 打开文件选择对话框
    // 判断选中的对话框不能为空
    if(m_path.isEmpty())
        QMessageBox::warning(this, "警告", "选中要发送的文件不能为空");
    // 将选中的文件路径显示到单行编辑框中
    ui->filePath_lineEdit->setText(m_path);
}
// 发送文件
void Widget::on_sendBtn_clicked()
{
    // 将选中的文件路径发送给任务类
    emit this->SendToFile(m_path);
}
```

程序运行结果：