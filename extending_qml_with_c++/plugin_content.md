# 插件内容（Plugin Content）

插件是一个已定义接口的库，它只在需要时才被加载。这与一个库在程序启动时被链接和加载不同。在QML场景下，这个接口叫做```QQmlExtensionPlugin```。我们关心其中的两个方法```initializeEngine()```和```registerTypes()```。当插件被加载时，首先会调用```initializeEngine()```，它允许我们访问引擎将插件对象暴露给根上下文。大多数时候你只会使用到```registerTypes()```方法。它允许你注册你自定义的QML类型到引擎提供的地址上。

我们稍微退一步考虑一个潜在的文件IO类型，它允许我们在QML中读取/写入一个小型文本文件。第一次的迭代可能看起来像在嘲笑QML的实现。

```
// FileIO.qml (good)
QtObject {
    function write(path, text) {};
    function read(path) { return "TEXT"}
}
```

这是一个纯粹的qml可能的实现，C++基于QML编程接口来探索一些编程接口。我们看到我们有一个读取和写入函数。写入函数需要一个路径和一个文本，读取函数需要一个路径，返回一个文本。路径和文本看起来是公共参数，或许我们可以将它们提取作为属性。

```
// FileIO.qml (better)
QtObject {
    property url source
    property string text
    function write() { // open file and write text };
    function read() { // read file and assign to text };
}
```

当然这看起来更像一个QML编程接口。我们使用属性让我们的环境能够绑定我们的属性并且响应变化。

在C++中创建这个编程接口我们需要创建类似的一个接口。

```
class FileIO : public QObject {
    ...
    Q_PROPERTY(QUrl source READ source WRITE setSource NOTIFY sourceChanged)
    Q_PROPERTY(QString text READ text WRITE setText NOTIFY textChanged)
    ...
public:
    Q_INVOKABLE void read();
    Q_INVOKABLE void write();
    ...
}
```

QML引擎需要注册```FileIO```类型。我们想要在```org.example.io```模块中使用它。

```
import org.example.io 1.0

FileIO {
}
```

一个插件可以在相同的模块中暴露若干个类型。但是不能从一个插件中暴露若干个模块。所以模块与插件之间的关系是一对一的。这个关系由模块标识符表示。
