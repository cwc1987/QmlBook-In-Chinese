# 间接连接（Connecting Indirectly）

动态创建QML元素时，无法使用onSignalName静态配置来连接信号。必须使用连接元素（Connection element）来完成连接信号。它可以连接一个目标元素任意数量的信号。

通过设置连接元素（Connection element）的目标属性，信号可以像正常的方法连接。也就是使用onSignalName方法。不管怎样，通过改变目标属性可以在不同的时间监控不同的元素。

![](http://qmlbook.github.io/_images/connections.png)

在上面这个例子中，用户界面由两个可点击区域组成。当其中一个区域点击后，会使用一个闪烁的动画。左边区域的代码段如下所示。在鼠标区域（MouseArea）中，左点击动画（leftClickedAnimation）被触发，导致区域闪烁。

```
        Rectangle {
            id: leftRectangle

            width: 290
            height: 200

            color: "green"

            MouseArea {
                id: leftMouseArea
                anchors.fill: parent
                onClicked: leftClickedAnimation.start();
            }

            Text {
                anchors.centerIn: parent
                font.pixelSize: 30
                color: "white"
                text: "Click me!"
            }
        }
```

除了两个可点击区域，还使用了一个连接元素（Connection element）。当状态为激活时会触发第三个动画，即元素的目标。

```
    Connections {
        id: connections
        onClicked: activeClickedAnimation.start();
    }
```
为了确定鼠标区域的目标，定义了两种状态。注意我们无法使用属性改变元素（PropertyChanges element）来设置目标属性，因为它已经包含了一个目标属性。利用状态改变脚本（StateChangeScript）来完成。

```
    states: [
        State {
            name: "left"
            StateChangeScript {
                script: connections.target = leftMouseArea
            }
        },
        State {
            name: "right"
            StateChangeScript {
                script: connections.target = rightMouseArea
            }
        }
    ]
```

当尝试运行这个例子时，需要注意当多个信号被处理调用所有操作时，执行的顺序是未定义的。

当创建一个连接元素（Connection element）未指定目标属性时，默认的属性是父对象。这意味着需要显式的设置NULL来避免捕获来自父对象的信号，直到目标被设置。这种行为使得基于连接元素（Connection element）创建自定义信号处理组件成为可能。使用这种方式可以将信号的处理代码封装和再使用。

在下面这个例子中，闪烁组件能够被放在任何的鼠标区域（MouseArea）中.点击后会触发动画，导致父对象闪烁。在同一个鼠标区域（MouseArea）的实际任务被触发时也可以被调用。这从实际的动作中，分离了标准的用户反馈，闪烁。

```
import QtQuick 2.0

Connections {
	onClicked: {
		// Automatically targets the parent
	}
}
```

只需要简单的在每个鼠标区域（MouseArea）实例化一个闪烁组件来实现闪烁。

```
import QtQuick 2.0

Item {
	// A background flasher that flashes the background of any parent MouseArea
}
```

当你使用一个连接元素（Connection element）来监控不同类型的目标元素的信号时，你可能会发现在在某些场景下会有来自不同目标的可用信号。这将导致连接元素（Connections element）由于丢失信号输出运行错误（run-time errors）。为了避免这个问题，设置忽略未知信号（ignoreUnknownSignal）属性为true，可以忽略这些错误。



