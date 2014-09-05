# 应用程序类型（Application Types）

这一节贯穿了可能使用Qt5编写的不同类型的应用程序。没有任何建议的选择，只是想告诉读者Qt5通常情况下能做些什么。

## 2.3.1 控制台应用程序

一个控制台应用程序不需要提供任何人机交互图形界面通常被称作系统服务，或者通过命令行来运行。Qt5附带了一系列现成的组件来帮助你非常有效的创建跨平台的控制台应用程序。例如网络应用程序编程接口或者文件应用程序编程接口，字符串的处理，自Qt5.1发布的高效的命令解析器。由于Qt是基于C++的高级应用程序接口，你能够快速的编程并且程序拥有快速的执行速度。不要认为Qt仅仅只是用户界面工具，它也提供了许多其它的功能。

**字符串处理**

在第一个例子中我们展示了怎样简单的增加两个字符串常量。这不是一个有用的应用程序，但能让你了解本地端C++应用程序没有事件循环时是什么样的。

```
// module or class includes
#include <QtCore>

// text stream is text-codec aware
QTextStream cout(stdout, QIODevice::WriteOnly);

int main(int argc, char** argv)
{
    // avoid compiler warnings
    Q_UNUSED(argc)
    Q_UNUSED(argv)
    QString s1("Paris");
    QString s2("London");
    // string concatenation
    QString s = s1 + " " + s2 + "!";
    cout << s << endl;
}
```

**容器类**

这个例子在应用程序中增加了一个链表和一个链表迭代器。Qt自带大量方便使用的容器类，并且其中的元素使用相同的应用程序接口模式。

```
QString s1("Hello");
QString s2("Qt");
QList<QString> list;
// stream into containers
list <<  s1 << s2;
// Java and STL like iterators
QListIterator<QString> iter(list);
while(iter.hasNext()) {
    cout << iter.next();
    if(iter.hasNext()) {
        cout << " ";
    }
}
cout << "!" << endl;
```

这里我们展示了一些高级的链表函数，允许你在一个字符串中加入一个链表的字符串。当你需要持续的文本输入时非常的方便。使用QString::split()函数可以将这个操作逆向（将字符串转换为字符串链表）。

```
QString s1("Hello");
QString s2("Qt");
// convenient container classes
QStringList list;
list <<  s1 << s2;
// join strings
QString s = list.join(" ") + "!";
cout << s << endl;
```

**文件IO**

下一个代码片段我们从本地读取了一个CSV文件并且遍历提取每一行的每一个单元的数据。我们从CSV文件中获取大约20行的编码。文件读取仅仅给了我们一个比特流，为了有效的将它转换为可以使用的Unicode文本，我们需要使用这个文件作为文本流的底层流数据。编写CSV文件，你只需要以写入的方式打开一个文件并且一行一行的输入到文件流中。

```
QList<QStringList> data;
// file operations
QFile file("sample.csv");
if(file.open(QIODevice::ReadOnly)) {
    QTextStream stream(&file);
    // loop forever macro
    forever {
        QString line = stream.readLine();
        // test for empty string 'QString("")'
        if(line.isEmpty()) {
            continue;
        }
        // test for null string 'String()'
        if(line.isNull()) {
            break;
        }
        QStringList row;
        // for each loop to iterate over containers
        foreach(const QString& cell, line.split(",")) {
            row.append(cell.trimmed());
        }
        data.append(row);
    }
}
// No cleanup necessary.
```

现在我们结束Qt关于基于控制台应用程序小节。

## 2.3.2 窗口应用程序

基于控制台的应用程序非常方便，但是有时候你需要有一些用户界面。但是基于用户界面的应用程序需要后端来写入/读取文件，使用网络进行通讯或者保存数据到一个容器中。

在第一个基于窗口的应用程序代码片段，我们仅仅只创建了一个窗口并显示它。没有父对象的窗口部件是Qt世界中的一个窗口。我们使用智能指针来确保当智能指针指向范围外时窗口会被删除掉。

这个应用程序对象封装了Qt的运行，调用exec开始我们的事件循环。从这里开始我们的应用程序只响应由鼠标或者键盘或者其它的例如网络或者文件IO的事件触发。应用程序也只有在事件循环退出时才退出，在应用程序中调用"quit()"或者关掉窗口来退出。
当你运行这段代码的时候你可以看到一个240乘以120像素的窗口。

```
#include <QtGui>

int main(int argc, char** argv)
{
    QApplication app(argc, argv);
    QScopedPointer<QWidget> widget(new CustomWidget());
    widget->resize(240, 120);
    widget->show();
    return app.exec();
}
```

**自定义窗口部件**

当你使用用户界面时你需要创建一个自定义的窗口部件。典型的窗口是一个窗口部件区域的绘制调用。附加一些窗口部件内部如何处理外部触发的键盘或者鼠标输入。为此我们需要继承QWidget并且重写几个函数来绘制和处理事件。

```
#ifndef CUSTOMWIDGET_H
#define CUSTOMWIDGET_H

#include <QtWidgets>

class CustomWidget : public QWidget
{
    Q_OBJECT
public:
    explicit CustomWidget(QWidget *parent = 0);
    void paintEvent(QPaintEvent *event);
    void mousePressEvent(QMouseEvent *event);
    void mouseMoveEvent(QMouseEvent *event);
private:
    QPoint m_lastPos;
};

#endif // CUSTOMWIDGET_H
```

在实现中我们绘制了窗口的边界并在鼠标最后的位置上绘制了一个小的矩形框。这是一个非常典型的低层次的自定义窗口部件。鼠标或者键盘事件会改变窗口的内部状态并触发重新绘制。我们不需要更加详细的分析这个代码，你应该有能力分析它。Qt自带了大量现成的桌面窗口部件，你有很大的几率不需要再做这些工作。

```
#include "customwidget.h"

CustomWidget::CustomWidget(QWidget *parent) :
    QWidget(parent)
{
}

void CustomWidget::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QRect r1 = rect().adjusted(10,10,-10,-10);
    painter.setPen(QColor("#33B5E5"));
    painter.drawRect(r1);

    QRect r2(QPoint(0,0),QSize(40,40));
    if(m_lastPos.isNull()) {
        r2.moveCenter(r1.center());
    } else {
        r2.moveCenter(m_lastPos);
    }
    painter.fillRect(r2, QColor("#FFBB33"));
}

void CustomWidget::mousePressEvent(QMouseEvent *event)
{
    m_lastPos = event->pos();
    update();
}

void CustomWidget::mouseMoveEvent(QMouseEvent *event)
{
    m_lastPos = event->pos();
    update();
}
```

**桌面窗口**

Qt的开发者们已经为你做好大量现成的桌面窗口部件，在不同的操作系统中他们看起来都像是本地的窗口部件。你的工作只需要在一个打的窗口容器中安排不同的的窗口部件。在Qt中一个窗口部件能够包含其它的窗口部件。这个操作由分配父子关系来完成。这意味着我们需要准备类似按钮（button），复选框（check box），单选按钮（radio button）的窗口部件并且对它们进行布局。下面展示了一种完成的方法。

这里有一个头文件就是所谓的窗口部件容器。

```
class CustomWidget : public QWidget
{
    Q_OBJECT
public:
    explicit CustomWidgetQWidget *parent = 0);
private slots:
    void itemClicked(QListWidgetItem* item);
    void updateItem();
private:
    QListWidget *m_widget;
    QLineEdit *m_edit;
    QPushButton *m_button;
};
```

在实现中我们使用布局来更好的安排我们的窗口部件。当容器窗口部件大小被改变后它会按照窗口部件的大小策略进行重新布局。在这个例子中我们有一个链表窗口部件，行编辑器与按钮垂直排列来编辑一个城市的链表。我们使用Qt的信号与槽来连接发送和接收对象。

```
CustomWidget::CustomWidget(QWidget *parent) :
    QWidget(parent)
{
    QVBoxLayout *layout = new QVBoxLayout(this);
    m_widget = new QListWidget(this);
    layout->addWidget(m_widget);

    m_edit = new QLineEdit(this);
    layout->addWidget(m_edit);

    m_button = new QPushButton("Quit", this);
    layout->addWidget(m_button);
    setLayout(layout);

    QStringList cities;
    cities << "Paris" << "London" << "Munich";
    foreach(const QString& city, cities) {
        m_widget->addItem(city);
    }

    connect(m_widget, SIGNAL(itemClicked(QListWidgetItem*)), this, SLOT(itemClicked(QListWidgetItem*)));
    connect(m_edit, SIGNAL(editingFinished()), this, SLOT(updateItem()));
    connect(m_button, SIGNAL(clicked()), qApp, SLOT(quit()));
}

void CustomWidget::itemClicked(QListWidgetItem *item)
{
    Q_ASSERT(item);
    m_edit->setText(item->text());
}

void CustomWidget::updateItem()
{
    QListWidgetItem* item = m_widget->currentItem();
    if(item) {
        item->setText(m_edit->text());
    }
}
```

**绘制图形**

有一些问题最好用可视化的方式表达。如果手边的问题看起来有点像几何对象，qt graphics view是一个很好的选择。一个图形视窗（graphics view）能够在一个场景（scene）排列简单的几何图形。用户可以与这些图形交互，它们使用一定的算法放置在场景（scene）上。填充一个图形视图你需要一个图形窗口（graphics view）和一个图形场景（graphics scene）。一个图形场景（scene）连接在一个图形窗口（view）上，图形对象（graphics item）是被放在图形场景（scene）上的。这里有一个简单的例子，首先头文件定义了图形窗口（view）与图形场景（scene）。

```
class CustomWidgetV2 : public QWidget
{
    Q_OBJECT
public:
    explicit CustomWidgetV2(QWidget *parent = 0);
private:
    QGraphicsView *m_view;
    QGraphicsScene *m_scene;

};
```

在实现中首先将图形场景（scene）与图形窗口（view）连接。图形窗口（view）是一个窗口部件，能够被我们的窗口部件容器包含。最后我们添加一个小的矩形框在图形场景（scene）中。然后它会被渲染到我们的图形窗口（view）上。

```
#include "customwidgetv2.h"

CustomWidget::CustomWidget(QWidget *parent) :
    QWidget(parent)
{
    m_view = new QGraphicsView(this);
    m_scene = new QGraphicsScene(this);
    m_view->setScene(m_scene);

    QVBoxLayout *layout = new QVBoxLayout(this);
    layout->setMargin(0);
    layout->addWidget(m_view);
    setLayout(layout);

    QGraphicsItem* rect1 = m_scene->addRect(0,0, 40, 40, Qt::NoPen, QColor("#FFBB33"));
    rect1->setFlags(QGraphicsItem::ItemIsFocusable|QGraphicsItem::ItemIsMovable);
}
```

## 2.3.3 数据适配

到现在我们已经知道了大多数的基本数据类型，并且知道如何使用窗口部件和图形视图（graphics views）。通常在应用程序中你需要处理大量的结构体数据，也可能需要不停的储存它们，或者这些数据需要被用来显示。对于这些Qt使用了模型的概念。下面一个简单的模型是字符串链表模型，它被一大堆字符串填满然后与一个链表视图（list view）连接。

```
m_view = new QListView(this);
m_model = new QStringListModel(this);
view->setModel(m_model);

QList<QString> cities;
cities << "Munich" << "Paris" << "London";
model->setStringList(cities);
```

另一个比较普遍的用法是使用SQL（结构化数据查询语言）来存储和读取数据。Qt自身附带了嵌入式版的SQLLite并且也支持其它的数据引擎（比如MySQL，PostgresSQL，等等）。首先你需要使用一个模式来创建你的数据库，比如像这样：

```
CREATE TABLE city (name TEXT, country TEXT);
INSERT INTO city value ("Munich", "Germany");
INSERT INTO city value ("Paris", "France");
INSERT INTO city value ("London", "United Kingdom");
```

为了能够在使用sql，我们需要在我们的项目文件（*.pro）中加入sql模块。

```
QT += sql
```

然后我们需要c++来打开我们的数据库。首先我们需要获取一个指定的数据库引擎的数据对象。使用这个数据库对象我们可以打开数据库。对于SQLLite这样的数据库我们可以指定一个数据库文件的路径。Qt提供了一些高级的数据库模型，其中有一种表格模型（table model）使用表格标示符和一个选项分支语句（where clause）来选择数据。这个模型的结果能够与一个链表视图连接，就像之前连接其它数据模型一样。

```
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
db.setDatabaseName('cities.db');
if(!db.open()) {
    qFatal("unable to open database");
}

m_model = QSqlTableModel(this);
m_model->setTable("city");
m_model->setHeaderData(0, Qt::Horizontal, "City");
m_model->setHeaderData(1, Qt::Horizontal, "Country");

view->setModel(m_model);
m_model->select();
```

对高级的模型操作，Qt提供了一种分类文件代理模型，允许你使用基础的分类排序和数据过滤来操作其它的模型。

```
QSortFilterProxyModel* proxy = new QSortFilterProxyModel(this);
proxy->setSourceModel(m_model);
view->setModel(proxy);
view->setSortingEnabled(true);
```

数据过滤基于列号与一个字符串参数完成。

```
proxy->setFilterKeyColumn(0);
proxy->setFilterCaseSensitive(Qt::CaseInsensitive);
proxy->setFilterFixedString(QString)
```

过滤代理模型比这里演示的要强大的多，现在我们只需要知道有它的存在就够了。

**注意**

**这里是综述了你可以在Qt5中开发的不同类型的经典应用程序。桌面应用程序正在发生着改变，不久之后移动设备将会为占据我们的世界。移动设备的用户界面设计非常不同。它们相对于桌面应用程序更加简洁，只需要专注的做一件事情。动画效果是一个非常重要的部分，用户界面需要生动活泼。传统的Qt技术已经不适于这些市场了。**

**接下来：Qt Quick将会解决这个问题。**

## 2.3.4 Qt Quick应用程序

在现代的软件开发中有一个内在的冲突，用户界面的改变速度远远高于我们的后端服务。在传统的技术中我们开发的前端需要与后端保持相同的步调。当一个项目在开发时用户想要改变用户界面，或者在一个项目中开发一个用户界面的想法就会引发这个冲突。敏捷项目需要敏捷的方法。

Qt Quick 提供了一个类似HTML声明语言的环境应用程序作为你的用户界面前端（the front-end），在你的后端使用本地的c++代码。这样允许你在两端都游刃有余。

下面是一个简单的Qt Quick UI的例子。

```
import QtQuick 2.0

Rectangle {
    width: 240; height: 1230
    Rectangle {
        width: 40; height: 40
        anchors.centerIn: parent
        color: '#FFBB33'
    }
}
```

这种声明语言被称作QML，它需要在运行时启动。Qt提供了一个典型的运行环境叫做qmlscene，但是想要写一个自定义的允许环境也不是很困难，我们需要一个快速视图（quick view）并且将QML文档作为它的资源。剩下的事情就只是展示我们的用户界面了。

```
QQuickView* view = new QQuickView();
QUrl source = Qurl::fromLocalUrl("main.qml");
view->setSource(source);
view.show();
```

回到我们之前的例子，在一个例子中我们使用了一个c++的城市数据模型。如果我们能够在QML代码中使用它将会更加的好。

为了实现它我们首先要编写前端代码怎样展示我们需要使用的城市数据模型。在这一个例子中前端指定了一个对象叫做cityModel，我们可以在链表视图（list view）中使用它。

```
import QtQuick 2.0

Rectangle {
    width: 240; height: 120
    ListView {
        width: 180; height: 120
        anchors.centerIn: parent
        model: cityModel
        delegate: Text { text: model.city }
    }
}
```

为了使用cityModel，我们通常需要重复使用我们以前的数据模型，给我们的根环境（root context）加上一个内容属性（context property）。（root context是在另一个文档的根元素中）。

```
m_model = QSqlTableModel(this);
... // some magic code
QHash<int, QByteArray> roles;
roles[Qt::UserRole+1] = "city";
roles[Qt::UserRole+2] = "country";
m_model->setRoleNames(roles);
view->rootContext()->setContextProperty("cityModel", m_model);
```

**警告**

**这不是完全正确的用法，作为包含在SQL表格模型列中的数据，一个QML模型的任务是来表达这些数据。所以需要做一个在列和任务之间的映射关系。请查看来[QML and QSqlTableModel](http://qt-project.org/wiki/QML_and_QSqlTableModel)获得更多的信息。**
