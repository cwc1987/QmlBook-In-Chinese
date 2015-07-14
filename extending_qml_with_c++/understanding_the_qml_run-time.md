# 理解QML运行环境（Understanding the QML Run-time）

当运行QML时，它在一个运行时环境下执行。这个运行时环境是由```QtQml```模块下的C++代码实现的。它由一个负责执行QML的引擎，持有访问每个组件属性的上下文和实例化的QML元素组件构成。

```
#include <QtGui>
#include <QtQml>

int main(int argc, char **argv)
{
    QGuiApplication app(argc, argv);
    QUrl source(QStringLiteral("qrc:/main.qml"));
    QQmlApplicationEngine engine;
    engine.load(source);
    return app.exec();
}
```

在这个例子中，```QGuiApplication```封装了所有与应用程序引用相关的属性（例如应用程序名称，命令行参数，和事件循环管理）。```QQmlApplicationEngine```分层管理上下文和组件的顺序。它需要加载一个典型的qml文件作为应用程序的开始点。在这个例子中，```main.qml```包含了以一个窗口和一个文本。

**注意**

**通过```QmlApplicationEngine```加载一个使用简单项作为根类型的```main.qml```不会在你的屏幕上显示任何东西，它需要一个窗口来管理一个平面的渲染。引擎可以加载不包含任何用户界面的qml代码（例如一个纯粹的对象）。由于它不会默认为你创建一个窗口。```qmlcene```或者新的qml运行环境将会在内部首先检查```main.qml```文件是否包含一个窗口作为根项，如果没有包含将会为你创建一个并且设置根项作为新创建窗口的子项。**

```
import QtQuick 2.4
import QtQuick.Window 2.0

Window {
    visible: true
    width: 512
    height: 300

    Text {
        anchors.centerIn: parent
        text: "Hello World!"
    }
}
```

在QML文件中我们定义我们的依赖是```QtQuick```和```QtQuick.Window```。这些定义将会触发在导入的路径中查找这些模块，并在加载成功后由引擎加载需要的插件。新加载的类型将会倍qmldir控制在qml文件中可用。

当然也可以使用快速创建插件直接向引擎添加我们的自定义类型。这里我们假设我们有一个基于```QObject```的```CurrentTime```类。

```
QQmlApplicationEngine engine;

qmlRegisterType<CurrentTime>("org.example", 1, 0, "CurrentTime");

engine.load(source);
```

现在我们也可可以在我们的qml文件中使用```CurrentTime```类型。

```
import org.example 1.0

CurrentTime {
    // access properties, functions, signals
}
```

一种更偷懒的方式是通过上下文属性直接设置。

```
QScopedPointer<CurrentTime> current(new CurrentTime());

QQmlApplicationEngine engine;

engine.rootContext().setContextProperty("current", current.value())

engine.load(source);
```

**注意**

**不要混淆```setContextProperty()```和setProperty()。```setContextProperty()```是设置一个qml上下文的属性，```setProperty()```是设置一个QObject的动态属性值，这对你没什么帮助。**

现在你可以在你的应用程序的任何地方使用这个属性了。感谢上下文继承这一特性。

```
import QtQuick 2.4
import QtQuick.Window 2.0

Window {
    visible: true
    width: 512
    height: 300

    Component.onCompleted: {
        console.log('current: ' + current)
    }
}
```

通常有以下几种不同的方式扩展QML：

* 上下文属性 - setContextProperty()

* 引擎注册类型 - 在main.cpp中调用qmlRegisterType

* QML扩展插件 - 后面会讨论

上下文属性使用对于小型的应用程序使用非常方便。它们不需要你做太多的事情就可以将系统编程接口暴露为友好的全局对象。它有助于确保不会出现命名冲突（例如使用（$）这种特殊符号，例如$.currentTime）。在JS变量中$是一个有效的字符。

注册QML类型允许用户从QML中控制一个c++对象的生命周期。上下文属性无法完成这间事情。它也不会破坏全局命名空间。所有的类型都需要先注册，并且在程序启动时会链接所有库，这在大多数情况下都不是一个问题。

QML扩展插件提供了最灵活的扩展系统。它允许你在插件中注册类型，在第一个QML文件调用导入鉴定时会加载这个插件。由于使用了一个QML单例这也不会再破坏全局命名空间。插件允许你跨项目重用模块，这在你使用Qt包含多个项目时非常方便。

这章的其余部分将会集中在qml扩展插件上讨论。它们提供了最好的灵活性和可重用性。
