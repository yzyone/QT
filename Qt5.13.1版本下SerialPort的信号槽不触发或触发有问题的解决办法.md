# Qt5.13.1版本下SerialPort的信号槽&QSerialPort::readyRead 不触发或触发有问题的解决办法

   上周想测试写qt的串口功能，搭建了界面，编写了串口收发测试测函数，结果发现，qt串口接收，或者发送遇到问题。

主要是两个问题。1.使用qt的串口发送数据，数据只发送了一次的问题。2.使用虚拟串口给qt串口发数据，无法触发readyRead信号，从而无法触发槽函数的问题。

这两个问题花了一些时间解决，网上搜索发现是Qt5.13.1的Bug。通过这篇文章，找到了解决办法，经过测试，发现可行。

[如何使Qt5.13.1的QSerialPort工作？ | 经验摘录 (1r1g.com)](https://qa.1r1g.com/sf/ask/4061446441/)

如下图是我设计的简单界面，功能还未完善，但具备了串口的打开，关闭，功能，内部使用了qDebug() 来打印是否收到了数据。 并使用 点击按钮的方式来发送helloWorld。

![](E:\GitHub\QT\serialport\26f2511d8f804edb9ecb0d15a8653a65.png)

**1.简单串口界面** 

我使用的是Qt5.13.1+VS2019开发的，需要在VS的：菜单栏>>xxx调试属性中添加QSerialPort模块，如下图所示：

![](E:\GitHub\QT\serialport\7c11cc047aa64eab94d3503143545c08.png)

 **2.添加serialport模块**

3.在串口配置好后，并打开，与外部串口或虚拟串口使用相同的串口参数（波特率，停止位，校验位，数据位等）.. m_serialport 是QSerialPort 的中串口的一个实例化对象。由于串口发送的数据是*char类型，所以需要将字符串转换为对应的格式。

//点击发送按钮,测试是否发送数据到其他串口助手上
```
void SerialPortForm::on_sendDataPushButton_clicked()
{
	if (isSerialOpen)
	{
		m_serialport->write(QString("Hello world").toLatin1());
		//m_serialport->waitForBytesWritten(100);//等待ms后。有数据后阻塞一段时间。
	}
}
```

点击按钮后，在其他串口助手上，得到hello world.结果如图：

![](E:\GitHub\QT\serialport\81531994416d4734b45df196be0885d0.png)

 **3.收到串口发来的数据**

在其他串口助手上点击发送hello，qt的程序收到了hello的结果。

```
//读取串口数据的槽函数,在实例化对象后，需要进行如下的信号槽连接，则有数据后能触发串口
//connect(this, &QSerialPort::readyRead, this, &MySerialPort::SerialRecieveData);
void MySerialPort::SerialRecieveData()
{
	QString framedata;
	/*读取串口收到的数据*/
	QByteArray bytedata = this->readAll();
	qDebug() << QString(bytedata);
}
```

![](E:\GitHub\QT\serialport\9161f1bb9751466bb80fefa3f3edc364.png)

**4.qt收到hello字符串**

以上串口收发的简单测试成功。

**Qt5.13.1的串口的bug修改**

 1.找到Qt的安装目录C:\Qt\Qt5.13.1\5.13.1\Src\qtserialport ； 找到qtserialport.pro，用QtCreator打开。如果是用VS开发，就要使用MSVC 2017构建来运行。 如果不是用VS，则用Qt默认的构建即可。

2.在项目树中打开 src/serialport/serialport-lib/sources/qwinoverlappedionotifier.cpp

3.将void QWinOverlappedIoNotifierPrivate::notify(DWORD numberOfBytes, DWORD errorCode,
        OVERLAPPED *overlapped)函数更改为如下：

```
/*
 * Note: This function runs in the I/O completion port thread.
 */
    void QWinOverlappedIoNotifierPrivate::notify(DWORD numberOfBytes, DWORD errorCode,
        OVERLAPPED *overlapped)
    {
    Q_Q(QWinOverlappedIoNotifier);
    WaitForSingleObject(hResultsMutex, INFINITE);
    results.enqueue(IOResult(numberOfBytes, errorCode, overlapped));
    ReleaseMutex(hResultsMutex);
    ReleaseSemaphore(hSemaphore, 1, NULL);
    //    if (!waiting && pendingNotifications-- == 0)
    //        emit q->_q_notify();
    if(!waiting)
        emit q->_q_notify();
    }
```

4.将void QWinOverlappedIoNotifierPrivate::_q_notified()更改如下

```
void QWinOverlappedIoNotifierPrivate::_q_notified()
{
    //int n = pendingNotifications.fetchAndStoreAcquire(0);
   // while (--n >= 0) {
        if (WaitForSingleObject(hSemaphore, 0) == WAIT_OBJECT_0)
            dispatchNextIoResult();
    //}
}
```

5.保存，清理项目，并重新编译所有项目。

在\Qt5.13.1\5.13.1\Src打开编译后的文件夹build-qtserialport-Desktop_Qt_5_13_1_MinGW_64_bit-Debug

或build-qtserialport-Desktop_Qt_5_13_1_MSVC2017_64bit-Debug

将对应的bin文件夹下的Qt5SerialPort.dll ，Qt5SerialPortd.dll 复制替换到\mingw73_64\bin   

  或者\msvc2017_64\bin中。

将编译生成的lib文件中的4个文件复制到msvc2017_64\lib中，即可解决5.13.1串口问题。（_MinGW_64_bit为于它同样编译器对应的lib文件）

![](E:\GitHub\QT\serialport\e4b36a6f0044496e8a0f893c87f19f76.png)

 问题解决。

————————————————

''版权声明：本文为CSDN博主「zeal_for_rov」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/zeal_for_rov/article/details/121739960