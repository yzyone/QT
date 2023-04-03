# Qt 嵌入浏览器 QWebEngineView实现浏览器基本功能

2022-12-28 15:25·[QT高级进阶](https://www.toutiao.com/c/user/token/MS4wLjABAAAAt_BOijoaoSs32bWWP-4kn4w4vK6Ml7vxKe16MMXkZiuF3N4YH95xKOtz9OB6zOVG/?source=tuwen_detail)

# 本篇简介

本篇的小目标：

- 借助Qt自家的QWebEngineView实现浏览器的基本功能：输入地址访问页面和刷新页面
- 定制QWebEngineView的ContextMenu，实现Inspector调试界面的调用

# QWebEngineView基础

首先在所创建项目的.pro配置中添加webenginewidgets模块：

```
QT += webenginewidgets
```

然后在主窗口初始化时创建QWebEngineView对象：

```
1 m_webView = new QWebEngineView(this);
2 QStackedLayout* layout = new QStackedLayout(ui->frame);
3 ui->frame->setLayout(layout);
4 layout->addWidget(m_webView);
```

界面上有一个输入地址的控件(adressEdit)和两个按钮——访问按钮（btnGo）和刷新按钮(btnRefresh)，使用QWebEngineView的load和reload方法，可以很方便地实现这两个按钮的功能：

```
 1 connect(ui->btnGo, &QPushButton::clicked, this, [this]() {
 2     QString url = ui->addressEdit->text();
 3     if (!url.isEmpty())
 4     {
 5         m_webView->load(url);
 6     }
 7 });
 8 connect(ui->btnRefresh, &QPushButton::clicked, this, [this]() {
 9     m_webView->reload();
10 });
```

这样一个简单的浏览器就实现好了，访问一下百度看看效果：

![img](./web/7112d15d82e04df4873c773634a70201~noop.image)



# 实现Inspector调试界面

在谷歌浏览器中按一下F12可以调出功能强大的调试界面，QWebEngine中也包含了这个功能。这里我们稍微简化一下，改成在页面上点击右键并选择"Inspect"，即可呼出调试界面。

**【领QT开发教程学习资料，点击下方链接莬费领取↓↓，先码住不迷路~】**

点击 →领取[「链接」](http://docs.qq.com/doc/DQlVKZG14U1hCQXJu)

首先需要设置一个环境变量
QTWEBENGINE_REMOTE_DEBUGGING来指定调试页面所使用的端口号。例如，将7777端口设为调试端口，可在主窗口初始化方法的最开头添加下面的代码：

```
qputenv("QTWEBENGINE_REMOTE_DEBUGGING", "7777");
```

如果设置成功，在终端上会打印如下提示：

```
Remote debugging server started successfully. Try pointing a Chromium-based browser to http://127.0.0.1:7777
```

然后实现一个QDialog作为Inspector的界面，里面内嵌另一个QWebEngineView，这个view专门用来加载调试页面：

```
 1 Inspector::Inspector(QWidget *parent) :
 2     QDialog(parent),
 3     ui(new Ui::Inspector)
 4 {
 5     ui->setupUi(this);
 6 
 7     connect(ui->btnClose, &QPushButton::clicked, this, [this](){
 8         hide();
 9     });
10 
11     m_webView = new QWebEngineView(this);
12     QStackedLayout* layout = new QStackedLayout(ui->frame);
13     ui->frame->setLayout(layout);
14     layout->addWidget(m_webView);
15     m_webView->load(QUrl("http://localhost:7777"));
16     QDialog::show();
17 }
```

因为这里的关闭按钮实际上只是把界面隐藏起来了，所以重载一下show方法，保证每次打开时调试的页面是最新的：

```
1 void Inspector::show()
2 {
3     m_webView->reload();
4     QDialog::show();
5 }
```

最后在主窗口初始化时修改一下QWebEngineViewContextMenu设置。因为QWebEngineView继承了QWidget，所以可以使用与处理QWidget类似的方式定制ContextMenu：

```
 1 m_webView->setContextMenuPolicy(Qt::CustomContextMenu);
 2 m_inspector = NULL;
 3 connect(m_webView, &QWidget::customContextMenuRequested, this, [this]() {
 4     QMenu* menu = new QMenu(this);
 5     QAction* action = menu->addAction("Inspect");
 6     connect(action, &QAction::triggered, this, [this](){
 7         if(m_inspector == NULL)
 8         {
 9             m_inspector = new Inspector(this);
10         }
11         else
12         {
13             m_inspector->show();
14         }
15     });
16     menu->exec(QCursor::pos());
17 });
```

这样一个简单的Inspector就实现完成了，试试效果：

![img](./web/59817f989f4a40ffbaf78e6b9f5fe171~noop.image)