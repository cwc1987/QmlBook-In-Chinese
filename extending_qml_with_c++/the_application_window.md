# 应用程序窗口（The Application Window）

使用Qt Creator的QtQuick Application向导创建一个基于QtQuick controls的应用程序。我们将不再使用新的QML格式，这在一本书里面将很难解释，即使新格式使用ui.qml文件将比之前更加容易达到目的。所以你可以移除/删除格式文件。

一个应用程序窗口基础配置包含了一个工具栏，菜单栏和状态栏。我们只使用菜单栏创建一些典型的菜单条目来打开和保存文档。基础配置的窗口只会显示一个空的窗口。

```
import QtQuick 2.4
import QtQuick.Controls 1.3
import QtQuick.Window 2.2
import QtQuick.Dialogs 1.2

ApplicationWindow {
    id: root
    title: qsTr("City UI")
    width: 640
    height: 480
    visible: true
}
```

