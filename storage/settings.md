# Settings

Qt自身就提供了基于系统方式的应用程序配置（又名选项，偏好）C++类 QSettings。它使用基于当前操作系统的方式存储配置。此外，它支持通用的INI文件格式用来操作跨平台的配置文件。

在Qt5.2中，配置（Settings）被加入到QML中。编程接口仍然在实验模块中，这意味着接口可能在未来会改变。这里需要注意。

这里有一个小例子，对一个矩形框配置颜色。每次用户点击窗口生成一个新的随机颜色。应用程序关闭后重启你将会看到你最后看到的颜色。
默认的颜色是用来初始化根矩形框的颜色。

```
import QtQuick 2.0
import Qt.labs.settings 1.0

Rectangle {
    id: root
    width: 320; height: 240
    color: '#000000'
    Settings {
        id: settings
        property alias color: root.color
    }
    MousArea {
        anchors.fill: parent
        onClicked: root.color = Qt.hsla(Math.random(), 0.5, 0.5, 1.0);
    }
}
```

每次颜色值的变化都被存储在配置中。这可能不是我们需要的。只有在要求使用标准属性的时候才存储配置。

```
Rectangle {
    id: root
    color: settings.color
    Settings {
        id: settings
        property color color: '#000000'
    }
    function storeSettings() { // executed maybe on destruction
        settings.color = root.color
    }
}
```

可以使用category属性存储不同种类的配置。

```
Settings {
    category: 'window'
    property alias x: window.x
    property alias y: window.x
    property alias width: window.width
    property alias height: window.height
}
```

配置同城根据你的应用程序名称，组织和域存储。这些信息通常在你的C++ main函数中设置。

```
int main(int argc, char** argv) {
    ...
    QCoreApplication::setApplicationName("Awesome Application");
    QCoreApplication::setOrganizationName("Awesome Company");
    QCoreApplication::setOrganizationDomain("org.awesome");
    ...
}
```


