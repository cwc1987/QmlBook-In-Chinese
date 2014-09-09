# 通过HTTP服务UI（Serving UI via HTTP）

通过HTTP加载一个简单的用户界面，我们需要一个web服务器，它为UI文件服务。但是首先我们需要有用户界面，我们在项目里创建一个创建了红色矩形框的main.qml。

```
// main.qml
import QtQuick 2.0

Rectangle {
    width: 320
    height: 320
    color: '#ff0000'
}
```

我们加载一段python脚本来提供这个文件：

```
$ cd <PROJECT>
# python -m SimpleHTTPServer 8080
```

现在我们可以通过[http://localhost:8000/main.qml](http://localhost:8000/main.qml)来访问，你可以像下面这样测试：

```
$ curl http://localhost:8000/main.qml
```

或者你可以用浏览器来访问。浏览器无法识别QML，并且无法通过文档来渲染。我们需要创建一个可以浏览QML文档的浏览器。为了渲染文档，我们需要指出qmlscene的位置。不幸的是qmlscene只能读取本地文件。我们为了突破这个限制，我们可以使用自己写的qmlscene或者使用QML动态加载。我们选择动态加载的方式。我们选择一个加载元素来加载远程的文档。

```
// remote.qml
import QtQuick 2.0

Loader {
    id: root
    source: 'http://localhost:8080/main2.qml'
    onLoaded: {
        root.width = item.width
        root.height = item.height
    }
}
```

我们现在可以使用qmlscene来加载remote.qml文档。这里仍然有一个小问题。加载器将会调整加载项的大小。我们的qmlscene需要适配大小。可以使用--resize-to-root选项来运行qmlscene。

```
$ qmlscene --resize-to-root remote.qml
```

按照root元素调整大小，告诉qmlscene按照root元素的大小调它的窗口大小。remote现在从本地服务器加载main.qml，并且可以自动调整加载的用户界面。方便且简单。

**注意**

**如果你不想使用一个本地服务器，你可以使用来自GitHub的gist服务。Gist是一个在线剪切板服务，就像PasteBin等等。可以在[https://gist.github.com](https://gist.github.com )下使用。我创建了一个简单的gist例子，地址是[https://gist.github.com/jryannel/7983492](https://gist.github.com/jryannel/7983492)。这将会返回一个绿色矩形框。由于gist连接提供的是HTML代码，我们需要连接一个/raw来读取原始文件而不是HTML代码。**

```
// remote.qml
import QtQuick 2.0

Loader {
    id: root
    source: 'https://gist.github.com/jryannel/7983492/raw'
    onLoaded: {
        root.width = item.width
        root.height = item.height
    }
}
```

从网络加载另一个文件，你只需要引用组件名。例如一个Button.qml，只要它们在同一个远程文件夹下就能够像正常一样访问。

## 11.1.1 网络组件（Networked Components）

我们做了一个小实验。我们在远程端添加一个按钮作为可以复用的组件。

```
- src/main.qml
- src/Button.qml
```

我们修改main.qml来使用button：

```
import QtQuick 2.0

Rectangle {
    width: 320
    height: 320
    color: '#ff0000'

    Button {
        anchors.centerIn: parent
        text: 'Click Me'
        onClicked: Qt.quit()
    }
}
```

再次加载我们的web服务器：

```
$ cd src
# python -m SimpleHTTPServer 8080
```

再次使用http加载远mainQML文件：

```
$ qmlscene --resize-to-root remote.qml
```

我们看到一个错误：

```
http://localhost:8080/main2.qml:11:5: Button is not a type
```

所以，在远程加载时，QML无法解决Button组件的问题。如果代码使用本地加载qmlscene src/main.qml，将不会有问题。Qt能够直接解析本地文件，并且检测哪些组件可用，但是使用http的远程访问没有“list-dir”函数。我们可以在main.qml中使用import声明来强制QML加载元素：

```
import "http://localhost:8080" as Remote

...

Remote.Button { ... }
```

再次运行qmlscene后，它将正常工作：

```
$ qmlscene --resize-to-root remote.qml
```

这是完整的代码：

```
// main2.qml
import QtQuick 2.0
import "http://localhost:8080" 1.0 as Remote

Rectangle {
    width: 320
    height: 320
    color: '#ff0000'

    Remote.Button {
        anchors.centerIn: parent
        text: 'Click Me'
        onClicked: Qt.quit()
    }
}
```

一个更好的选择是在服务器端使用qmldir文件来控制输出：

```
// qmldir
Button 1.0 Button.qml
```

然后更新main.qml：

```
import "http://localhost:8080" 1.0 as Remote

...

Remote.Button { ... }
```

当从本地文件系统使用组件时，它们的创建没有延迟。当组件通过网络加载时，它们的创建是异步的。创建时间的影响是未知的，当其它组件已经完成时，一个组件可能还没有完成加载。当通过网络加载组件时，需要考虑这些。
