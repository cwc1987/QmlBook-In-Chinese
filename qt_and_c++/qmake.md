# QMake
QMake是用来读取项目文件并生成编译文件的工具。项目文件记录了你的项目配置，扩展依赖库和源代码文件。最简单包含一个源代码文件的项目可能像这样：

```
// myproject.pro

SOURCES += main.cpp
```

我们编译了一个基于项目文件名称```myproject```的可执行程序。这个编译将只包含```main.cpp```源文件。默认情况下我们会为项目添加QtCore和QtGui模块。如果我们的项目是一个QML应用程序，我们需要添加QtQuick和QtQml到这个链表中：

```
// myproject.pro

QT += qml quick

SOURCES += main.cpp
```

现在编译文件知道与Qt的QtQml和QtQuick模块链接。QMake使用```=```，```+=` and ``-=```来指定，添加和移除选项链表中的元素。例如一个只有控制台的编译将不会依赖UI，你需要移除QtGui模块：

```
// myproject.pro

QT -= gui

SOURCES += main.cpp
```

当你期望编译一个库来替代一个应用程序时，你需要改变编译模板：

```
// myproject.pro
TEMPLATE = lib

QT -= gui

HEADERS += utils.h
SOURCES += utils.cpp
```

现在项目将使用```utils.h```头文件和```utils.cpp```文件编译为一个没有UI依赖的库。库的格式依赖于你当前编译项目所用的操作系统。

通常将会有更加复杂的配置并且需要编译个项目配置。qmake提供了```subdirs```模板。假设我们有一个mylib和一个myapp项目。我们的配置可能如下：

```
my.pro
mylib/mylib.pro
mylib/utils.h
mylib/utils.cpp
myapp/myapp.pro
myapp/main.cpp
```

我们已经知道了如何使用Mylib.pro和myapp.pro。my.pro作为一个包含项目文件配置如下：

```
// my.pro
TEMPLATE = subdirs

subdirs = mylib \
    myapp

myapp.depends = mylib
```

在项目文件中声明包含两个子项目```mylib```和```myapp```，```myapp```依赖于```mylib```。当你使用qmake为这个项目文件生成编译文件时，将会在每个项目文件对应的文件夹下生成一个编译文件。当你使用```my.pro```文件的makefile编译时，所有的子项目也会编译。

有时你需要基于你的配置在不同的平台上做不同的事情。qmake推荐使用域的概念来处理它。当一个配置选项设置为true时使一个域生效。

例如使用unix指定utils的实现可以像这样：

```
unix {
    SOURCES += utils_unix.cpp
} else {
    SOURCES += utils.cpp
}
```

这表示如果CONFIG变量包含了unix配置将使用对应域下的文件路径，否则使用其它的文件路径。一个典型的例子，移除mac下的应用绑定：

```
macx {
    CONFIG -= app_bundle
}
```

这将在mac下创建的应用程序不再是一个```.app```文件夹而是创建一个应用程序替换它。

基于QMake的项目通常在你开始编写Qt应用程序最好的选择。但是也有一些其它的选择。它们各有优势。我们将在下一小节中简短的讨论其它的选择。

**引用**

[QMake Manual](http://doc.qt.io/qt-5//qmake-manual.html) - qmake手册目录。

[QMake Language](http://doc.qt.io/qt-5//qmake-language.html) - 赋值，域和相关语法

[QMake Variables](http://doc.qt.io/qt-5//qmake-variable-reference.html) - TEMPLATE，CONFIG，QT等变量解释


