# 演示程序（A Boilerplate Application）

理解Qt最好的方法是从一个小的应用程序开始。这个简单的例子叫做“Hello World!”，使用unicode编码将字符串写入到一个文件中。

```
#include <QCoreApplication>
#include <QString>
#include <QFile>
#include <QDir>
#include <QTextStream>
#include <QDebug>


int main(int argc, char *argv[])
{
    QCoreApplication app(argc, argv);

    // prepare the message
    QString message("Hello World!");

    // prepare a file in the users home directory named out.txt
    QFile file(QDir::home().absoluteFilePath("out.txt"));
    // try to open the file in write mode
    if(!file.open(QIODevice::WriteOnly)) {
        qWarning() << "Can not open file with write access";
        return -1;
    }
    // as we handle text we need to use proper text codecs
    QTextStream stream(&file);
    // write message to file via the text stream
    stream << message;

    // do not start the eventloop as this would wait for external IO
    // app.exec();

    // no need to close file, closes automatically when scope ends
    return 0;
}
```

这个简单的例子演示了文件访问的使用和通过文本流使用文本编码将文本正确的写入到文件中。二进制数据的操作有一个跨平台的二进制流类叫做QDataStream。我们使用的不同类需要使用它们的类名包含。另一种是使用模块名和类名
例如```#include <QtCore/QFile>```。对于比较懒的人，有一个更加简单的方法是包含整个模块，使用```#include <QtCore>```。例如在QtCore中你可以在应用程序中使用很多通用的类，这没有UI依赖。查看[QtCore class list](http://doc.qt.io/qt-5/qtcore-module.html)或者[QtCore overview](http://doc.qt.io/qt-5/qtcore-index.html)获取更多的信息。

使用qmake和make来构建程序。QMake读取项目文件（project file）并生成一个Makefile供make使用。项目文件（project file）独立于平台，qmake会根据特定平台的设置应用一些规则来生成Makefile。在有特殊需求的项目中，项目文件也可以包含特定平台规则的平台作用域。下面是一个简单的项目文件（project file）例子。

```
# build an application
TEMPLATE = app

# use the core module and do not use the gui module
QT       += core
QT       -= gui

# name of the executable
TARGET = CoreApp

# allow console output
CONFIG   += console

# for mac remove the application bundling
macx {
    CONFIG   -= app_bundle
}

# sources to be build
SOURCES += main.cpp
```

我们不会再继续深入这个话题，只需要记住Qt项目会使用特定的项目文件（project file），qmake会根据这些项目文件和指定平台生成Makefile。

上面简单的例子只是在应用程序中写入文本。一个命令行工具这是不够的。对于一个用户界面我们需要一个事件循环来等待用户的输入并安排刷新绘制操作。下面这个相同的例子使用一个桌面按钮来触发写入。

令人惊奇的是我们的main.cpp依然很小。我们将代码移入到我们的类中，并使用信号槽（signal/slots）来连接用用户的输入，例如按钮点击。信号槽（signal/slot）机制通常需要一个对象，你很快就会看到。

```
#include <QtCore>
#include <QtGui>
#include <QtWidgets>
#include "mainwindow.h"


int main(int argc, char** argv)
{
    QApplication app(argc, argv);

    MainWindow win;
    win.resize(320, 240);
    win.setVisible(true);

    return app.exec();
}
```

在main函数中我们简单的创建了一个应用程序对象，并使用exec()开始事件循环。现在应用程序放在了事件循环中，并等待用户输入。

```
int main(int argc, char** argv)
{
    QApplication app(argc, argv); // init application

    // create the ui

    return app.exec(); // execute event loop
}
```

Qt提供了几种UI技术。这个例子中我们使用纯Qt C++的桌面窗口用户界面库。我们需要创建一个主窗口来放置一个触发功能的按钮，同事由主窗口来实现我们的核心功能，正如我们在上面例子上看到的。

![](http://qmlbook.github.io/_images/storecontent.png)

主窗口本身也是一个窗口，它是一个没有父对象的窗口。这与Qt如何定义用户界面为一个界面元素树类似。在这个例子中，主窗口是我们的根元素，按钮是主窗口上的一个子元素。

```
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QtWidgets>

class MainWindow : public QMainWindow
{
public:
    MainWindow(QWidget* parent=0);
    ~MainWindow();
public slots:
    void storeContent();
private:
    QPushButton *m_button;
};

#endif // MAINWINDOW_H
```

此外我们定义了一个公有槽函数```storeContent()```，当点击按钮时会调用这个函数。槽函数是一个C++方法，这个方法被注册到Qt的源对象系统中，可以被动态调用。

```
#include "mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    m_button = new QPushButton("Store Content", this);

    setCentralWidget(m_button);
    connect(m_button, &QPushButton::clicked, this, &MainWindow::storeContent);
}

MainWindow::~MainWindow()
{

}

void MainWindow::storeContent()
{
    qDebug() << "... store content";
    QString message("Hello World!");
    QFile file(QDir::home().absoluteFilePath("out.txt"));
    if(!file.open(QIODevice::WriteOnly)) {
        qWarning() << "Can not open file with write access";
        return;
    }
    QTextStream stream(&file);
    stream << message;
}
```

在主窗口中我们首先创建了一个按钮，并将```clicked()```信号与```storeContent()```槽连接起来。每点击信号发送时都会触发调用槽函数```storeContent()```。就是这么简单，通过信号与槽的机制实现了松耦合的对象通信。
