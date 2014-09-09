# 组件（Compontents）


一个组件是一个可以重复使用的元素，QML提供几种不同的方法来创建组件。但是目前我们只对其中一种方法进行讲解：一个文件就是一个基础组件。一个以文件为基础的组件在文件中创建了一个QML元素，并且将文件以元素类型来命名（例如Button.qml）。你可以像任何其它的QtQuick模块中使用元素一样来使用这个组件。在我们下面的例子中，你将会使用你的代码作为一个Button（按钮）来使用。

让我们来看看这个例子，我们创建了一个包含文本和鼠标区域的矩形框。它类似于一个简单的按钮，我们的目标就是让它足够简单。

```
    Rectangle { // our inlined button ui
        id: button
        x: 12; y: 12
        width: 116; height: 26
        color: "lightsteelblue"
        border.color: "slategrey"
        Text {
            anchors.centerIn: parent
            text: "Start"
        }
        MouseArea {
            anchors.fill: parent
            onClicked: {
                status.text = "Button clicked!"
            }
        }
    }

    Text { // text changes when button was clicked
        id: status
        x: 12; y: 76
        width: 116; height: 26
        text: "waiting ..."
        horizontalAlignment: Text.AlignHCenter
    }
```

用户界面将会看起来像下面这样。左边是初始化的状态，右边是按钮点击后的效果。

![](http://qmlbook.org/_images/button_waiting.png)

![](http://qmlbook.org/_images/button_clicked.png)

我们的目标是提取这个按钮作为一个可重复使用的组件。我们可以简单的考虑一下我们的按钮会有的哪些API（应用程序接口），你可以自己考虑一下你的按钮应该有些什么。下面是我考虑的结果：

```
// my ideal minimal API for a button
Button {
    text: "Click Me"
    onClicked: { // do something }
}
```

我想要使用text属性来设置文本，然后实现我们自己的点击操作。我也期望这个按钮有一个比较合适的初始化大小（例如width:240）。
为了完成我们的目标，我创建了一个Button.qml文件，并且将我们的代码拷贝了进去。我们在根级添加一个属性导出方便使用者修改它。

我们在根级导出了文本和点击信号。通常我们命名根元素为root让引用更加方便。我们使用了QML的alias（别名）的功能，它可以将内部嵌套的QML元素的属性导出到外面使用。有一点很重要，只有根级目录的属性才能够被其它文件的组件访问。

```
// Button.qml

import QtQuick 2.0

Rectangle {
    id: root
    // export button properties
    property alias text: label.text
    signal clicked

    width: 116; height: 26
    color: "lightsteelblue"
    border.color: "slategrey"

    Text {
        id: label
        anchors.centerIn: parent
        text: "Start"
    }
    MouseArea {
        anchors.fill: parent
        onClicked: {
            root.clicked()
        }
    }
}
```

使用我们新的Button元素只需要在我们的文件中简单的声明一下就可以了，之前的例子将会被简化。

```
    Button { // our Button component
        id: button
        x: 12; y: 12
        text: "Start"
        onClicked: {
            status.text = "Button clicked!"
        }
    }

    Text { // text changes when button was clicked
        id: status
        x: 12; y: 76
        width: 116; height: 26
        text: "waiting ..."
        horizontalAlignment: Text.AlignHCenter
    }
```

现在你可以在你的用户界面代码中随意的使用Button{ ...}来作为按钮了。一个真正的按钮将更加复杂，比如提供按键反馈或者添加一些装饰。

**注意**

**就个人而言，可以更进一步的使用基础元素对象（Item）作为根元素。这样可以防止用户改变我们设计的按钮的颜色，并且可以提供出更多相关控制的API（应用程序接口）。我们的目标是导出一个最小的API（应用程序接口）。实际上我们可以将根矩形框（Rectangle）替换为一个基础元素（Item），然后将一个矩形框（Rectangle）嵌套在这个根元素（root item）就可以完成了。**

```
Item {
    id: root
    Rectangle {
        anchors.fill parent
        color: "lightsteelblue"
        border.color: "slategrey"
    }
    ...
}
```

使用这项技术可以很简单的创建一系列可重用的组件。
