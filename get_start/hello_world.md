# 你好世界（Hello World）

为了测试你的安装，我们创建一个简单的应用程序hello world.打开Qt Creator并且创建一个Qt Quick UI Project（File->New File 或者 Project-> Qt Quick Project -> Qt Quick UI）并且给项目取名 HelloWorld。

**注意**

**Qt Creator集成开发环境允许你创建不同类型的应用程序。如果没有另外说明，我们都创建Qt Quick UI Project。**

**提示**

**一个典型的Qt Quick应用程序在运行时解释，与本地插件或者本地代码在运行时解释代码一样。对于才开始的我们不需要关注本地端的解释开发，只需要把注意力集中在Qt5运行时的方面。**

Qt Creator将会为我们创建几个文件。HellWorld.qmlproject文件是项目文件，保存了项目的配置信息。这个文件由Qt Creator管理，我们不需要编辑它。

另一个文件HelloWorld.qml保存我们应用程序的代码。打开它，并且尝试想想这个应用程序要做什么，然后再继续读下去。

```
// HelloWorld.qml

import QtQuick 2.0

Rectangle {
    width: 360
    height: 360
    Text {
        anchors.centerIn: parent
        text: "Hello World"
    }
    MouseArea {
        anchors.fill: parent
        onClicked: {
            Qt.quit();
        }
    }
}
```

HelloWorld.qml使用QML语言来编写。我们将在下一章更深入的讨论QML语言，现在只需要知道它描述了一系列有层次的用户界面。这个代码指定了显示一个360乘以360像素的一个矩形，矩形中间有一个“Hello World"的文本。鼠标区域覆盖了整个矩形，当用户点击它时，程序就会退出。

你自己可以运行这个应用程序，点击左边的运行或者从菜单选择select Bulid->Run。

如果一切顺利，你将看到下面这个窗口：

![](http://qmlbook.org/_images/example.png)

Qt 5似乎已经可以工作了，我们接着继续。

**建议**

**如果你是一个名系统集成人员，你会想要安装最新稳定的Qt版本，将这个Qt版本的源代码编译到你特定的目标机器上。**

**从头开始构建**

如果你想使用命令行的方式构建Qt5，你首先需要拷贝一个代码库并构建他。

```
git clone git://gitorious.org/qt/qt5.git
cd qt5
./init-repository
./configure -prefix $PWD/qtbase -opensource
make -j4
```

等待两杯咖啡的时间编译完成后，qtbase文件夹中将会出现可以使用的Qt5。任何饮料都好，不过我喜欢喝着咖啡等待最好的结果。

如果你想测试你的编译，只需简单的启动qtbase/bin/qmlscene并且选择一个QtQuick的例子运行，或者跟着我们进入下一章。

为了测试你的安装，我们创建了一个简单的hello world应用程序。创建一个空的qml文件example1.qml，使用你最喜爱的文本编辑器并且粘贴一下内容：

```
// HelloWorld.qml

import QtQuick 2.0

Rectangle {
    width: 360
    height: 360
    Text {
        anchors.centerIn: parent
        text: "Greetings from Qt5"
    }
    MouseArea {
        anchors.fill: parent
        onClicked: {
            Qt.quit();
        }
    }
}
```

你现在使用来自Qt5的默认运行环境来可以运行这个例子：

```
$ qtbase/bin/qmlscene
```
