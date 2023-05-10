# Qt怎么创建SQlite数据库

Qt 创建 SQlite数据库

```
void Widget::initDB()
{
    // 创建并打开数据库
    QSqlDatabase database;
    database = QSqlDatabase::addDatabase("QSQLITE");
//    qDebug() << QApplication::applicationDirPath(); // 获取应用程序当前目录

    database.setDatabaseName("test.sqlite3");
    if(!database.open())
    {
        qDebug() << "Error: Failed to connect database." << database.lastError();
    }
    else
    {
        qDebug() << "Succeed to connect database.";
    }

    // 创建表格
    QSqlQuery sql_query = database.exec("DROP TABLE student");
    // 先清空一下表，可按需添加此句
//    sql_query.exec("DROP TABLE student");
    // 创建表格student
    if(!sql_query.exec("create table student(UserId int primary key, UserName text, PassWord text)"))
    {
        qDebug() << "Error: Fail to create table." << sql_query.lastError();
    }
    else
    {
        qDebug() << "Table created!";
    }
    // 填充表
    if(!sql_query.exec("INSERT INTO student VALUES(1, 'AppleCai', '23')"))
    {
        qDebug() << "Error: Fail to create table." << sql_query.lastError();
    }
    else
    {
        qDebug() << "add one created!";
    }
    // 批量填充表
    QStringList names;
    names << "小A" << "小B" << "小C" << "小D" << "小E" << "小F" << "小G" << "小H" << "小I";

    QStringList password;
    password << "12" << "23" << "34" << "45" << "56" << "67" << "78" << "89" << "90";
    // 绑定关键字后才能进行操作
    sql_query.prepare("INSERT INTO student (UserId, UserName, PassWord) "
                      "VALUES (:UserId, :UserName, :PassWord)");
    qint8 i = 0;
    foreach (QString name, names) // 从names表里获取每个名字
    {
        sql_query.bindValue(":UserId", i+2); // 向绑定值里加入名字
        sql_query.bindValue(":UserName", name); // 成绩
        sql_query.bindValue(":PassWord", password[i]); // 班级
        if(!sql_query.exec())
        {
            qDebug() << "Error: Fail." << sql_query.lastError();
        }
        i++;
    }
    // 读取sqlite
    studentInfo tmp;
    QVector<studentInfo> infoVect; // 数据库缓存
    sql_query.exec("SELECT * FROM student WHERE UserId >= 5 AND UserId <= 9;");
    while (sql_query.next())
    {
        tmp.UserId = sql_query.value(0).toInt();
        tmp.UserName = sql_query.value(1).toString();
        tmp.Password = sql_query.value(2).toString();
        qDebug() << tmp.UserId << tmp.UserName << tmp.Password;
        infoVect.push_back(tmp);
    }
    qDebug("done");

    // 更改表中数据
    sql_query.prepare("UPDATE student SET PassWord = 'admin' WHERE UserName = 'AppleCai'");
    if(!sql_query.exec())
    {
        qDebug() << "Error: Fail." << sql_query.lastError();
    }

    // 删除表中数据
    sql_query.prepare("DELETE FROM student WHERE UserName = '小H'");
    if(!sql_query.exec())
    {
        qDebug() << "Error: Fail." << sql_query.lastError();
    }
}
QT开发交流+赀料君羊：661714027
```

![img](./sql/7eb1ddf768a447479558d4d89d47d23c.image)



以下是个人代码备份

> 这个代码是在qt写的，包含了数据库的创建和写入，但是我在项目准备直接在dataGrip把数据一键导入创建好数据库之后再用qt里面的sql语句读，所以就不需要这一部分了

```
#include "sqlitedatabase.h"

SqliteDatabase::SqliteDatabase()
{
    qDebug() << "hhh";
//    initPickNameDB();
}

void SqliteDatabase::initPickNameDB()
{
    // 创建并打开数据库
    QSqlDatabase database;
    database = QSqlDatabase::addDatabase("QSQLITE");
//    qDebug() << QApplication::applicationDirPath();

    database.setDatabaseName(QApplication::applicationDirPath() + "/CONFIG/" + "PickNameDB.sqlite3");
    if(!database.open())
    {
        qDebug() << "Error: Failed to connect database." << database.lastError();
    }
    else
    {
        qDebug() << "Succeed to connect database.";
    }

    // 创建表格 先清空一下表
    QSqlQuery sql_query = database.exec("DROP TABLE department");
    sql_query = database.exec("DROP TABLE person");

    if(!sql_query.exec("create table department (Id int primary key not null, "
                       "DeptName vchar(50) not null )"))
    {
        qDebug() << "Error: Fail to create department table." << sql_query.lastError();
    }
    else
    {
        qDebug() << "Department table created!";
    }
    if(!sql_query.exec("create table person (Id int primary key not null , "
                       "DeptID integer not null , "
                       "PerName vchar(50) not null, "
                       "foreign key(DeptID) references department (Id))"))
    {
        qDebug() << "Error: Fail to create person table." << sql_query.lastError();
    }
    // 填充表
//    sql_query.exec("insert into department (id, name) values (1, '办领导')");
//    sql_query.exec("insert into department (id, name) values (2, '综合处')");
//    sql_query.exec("insert into department (id, name) values (3, '政策法规处')");
//    sql_query.exec("insert into department (id, name) values (4, '机构改革处')");
//    sql_query.exec("insert into department (id, name) values (5, '党群政法行政机构编制管理处')");
//    sql_query.exec("insert into department (id, name) values (6, '政府行政机构编制管理处')");
//    sql_query.exec("insert into department (id, name) values (7, '市县行政机构编制管理处')");
//    sql_query.exec("insert into department (id, name) values (8, '事业机构编制管理处')");
//    sql_query.exec("insert into department (id, name) values (9, '事业单位登记管理处')");
//    sql_query.exec("insert into department (id, name) values (10, '机构编制监督检查处')");
//    sql_query.exec("insert into department (id, name) values (11, '人事处')");
//    sql_query.exec("insert into department (id, name) values (12, '机关党委')");
//    sql_query.exec("insert into department (id, name) values (13, '省机构编制电子政务中心')");
//    sql_query.exec("insert into department (id, name) values (14, '省机构编制研究中心')");

    // 批量填充表
    QStringList deptNames;
    deptNames << "办领导" << "综合处" << "政策法规处" << "机构改革处"
              << "党群政法行政机构编制管理处" << "政府行政机构编制管理处"
              << "市县行政机构编制管理处" << "事业机构编制管理处" << "事业单位登记管理处"
              << "机构编制监督检查处" << "人事处" << "机关党委"
              << "省机构编制电子政务中心" << "省机构编制研究中心";

    // 绑定关键字后才能进行操作
    sql_query.prepare("INSERT INTO department (Id, DeptName) "
                      "VALUES (:Id, :DeptName)");
    qint8 i = 0;
    foreach (QString deptName, deptNames)
    {
        sql_query.bindValue(":Id", i + 1);
        sql_query.bindValue(":DeptName", deptName);
        if(!sql_query.exec())
        {
            qDebug() << "Error: Fail." << sql_query.lastError();
        }
        i++;
    }
    // 读取sqlite
    department dept;
    QVector<department> tmpDept; // 数据库缓存
    sql_query.exec("SELECT * FROM ");
}
QT开发交流+赀料君羊：661714027
```