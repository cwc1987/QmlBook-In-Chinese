# 输入元素（Input Element）

我们已经使用过MouseArea（鼠标区域）作为鼠标输入元素。这里我们将更多的介绍关于键盘输入的一些东西。我们开始介绍文本编辑的元素：TextInput（文本输入）和TextEdit（文本编辑）。

## 4.7.1 文本输入（TextInput）

文本输入允许用户输入一行文本。这个元素支持使用正则表达式验证器来限制输入和输入掩码的模式设置。

```
// textinput.qml

import QtQuick 2.0

Rectangle {
    width: 200
    height: 80
    color: "linen"

    TextInput {
        id: input1
        x: 8; y: 8
        width: 96; height: 20
        focus: true
        text: "Text Input 1"
    }

    TextInput {
        id: input2
        x: 8; y: 36
        width: 96; height: 20
        text: "Text Input 2"
    }
}
```

![](http://qmlbook.org/_images/textinput.png)

用户可以通过点击TextInput来改变焦点。为了支持键盘改变焦点，我们可以使用KeyNavigation（按键向导）这个附加属性。

```
// textinput2.qml

import QtQuick 2.0

Rectangle {
    width: 200
    height: 80
    color: "linen"

    TextInput {
        id: input1
        x: 8; y: 8
        width: 96; height: 20
        focus: true
        text: "Text Input 1"
        KeyNavigation.tab: input2
    }

    TextInput {
        id: input2
        x: 8; y: 36
        width: 96; height: 20
        text: "Text Input 2"
        KeyNavigation.tab: input1
    }
}
```

KeyNavigation（按键向导）附加属性可以预先设置一个元素id绑定切换焦点的按键。

一个文本输入元素（text input element）只显示一个闪烁符和已经输入的文本。用户需要一些可见的修饰来鉴别这是一个输入元素，例如一个简单的矩形框。当你放置一个TextInput（文本输入）在一个元素中时，你需要确保其它的元素能够访问它导出的大多数属性。

我们提取这一段代码作为我们自己的组件，称作TLineEditV1用来重复使用。

```
// TLineEditV1.qml

import QtQuick 2.0

Rectangle {
    width: 96; height: input.height + 8
    color: "lightsteelblue"
    border.color: "gray"

    property alias text: input.text
    property alias input: input

    TextInput {
        id: input
        anchors.fill: parent
        anchors.margins: 4
        focus: true
    }
}
```

**注意**

**如果你想要完整的导出TextInput元素，你可以使用property alias input: input来导出这个元素。第一个input是属性名字，第二个input是元素id。**

我们使用TLineEditV1组件重写了我们的KeyNavigation（按键向导）的例子。

```
    Rectangle {
        ...
        TLineEditV1 {
            id: input1
            ...
        }
        TLineEditV1 {
            id: input2
            ...
        }
    }
```

![](http://qmlbook.org/_images/textinput3.png)

尝试使用Tab按键来导航，你会发现焦点无法切换到input2上。这个例子中使用focus:true的方法不正确，这个问题是因为焦点被转移到input2元素时，包含TlineEditV1的顶部元素接收了这个焦点并且没有将焦点转发给TextInput（文本输入）。为了防止这个问题，QML提供了FocusScope（焦点区域）。

## 4.7.2 焦点区域（FocusScope）

一个焦点区域（focus scope）定义了如果焦点区域接收到焦点，它的最后一个使用focus:true的子元素接收焦点，它将会把焦点传递给最后申请焦点的子元素。我们创建了第二个版本的TLineEdit组件，称作TLineEditV2，使用焦点区域（focus scope）作为根元素。

```
// TLineEditV2.qml

import QtQuick 2.0

FocusScope {
    width: 96; height: input.height + 8
    Rectangle {
        anchors.fill: parent
        color: "lightsteelblue"
        border.color: "gray"

    }

    property alias text: input.text
    property alias input: input

    TextInput {
        id: input
        anchors.fill: parent
        anchors.margins: 4
        focus: true
    }
}
```

现在我们的例子将像下面这样：

```
    Rectangle {
        ...
        TLineEditV2 {
            id: input1
            ...
        }
        TLineEditV2 {
            id: input2
            ...
        }
    }
```

按下Tab按键可以成功的在两个组件之间切换焦点，并且能够正确的将焦点锁定在组件内部的子元素中。

## 4.7.3 文本编辑（TextEdit）

文本编辑（TextEdit）元素与文本输入（TextInput）非常类似，它支持多行文本编辑。它不再支持文本输入的限制，但是提供了已绘制文本的大小查询（paintedHeight，paintedWidth）。我们也创建了一个我们自己的组件TTextEdit，可以编辑它的背景，使用focus scope（焦点区域）来更好的切换焦点。

```
// TTextEdit.qml

import QtQuick 2.0

FocusScope {
    width: 96; height: 96
    Rectangle {
        anchors.fill: parent
        color: "lightsteelblue"
        border.color: "gray"

    }

    property alias text: input.text
    property alias input: input

    TextEdit {
        id: input
        anchors.fill: parent
        anchors.margins: 4
        focus: true
    }
}
```

你可以像下面这样使用这个组件：

```
// textedit.qml

import QtQuick 2.0

Rectangle {
    width: 136
    height: 120
    color: "linen"

    TTextEdit {
        id: input
        x: 8; y: 8
        width: 120; height: 104
        focus: true
        text: "Text Edit"
    }
}
```

![](http://qmlbook.org/_images/textedit.png)

## 4.7.4 按键元素（Key Element）

附加属性key允许你基于某个按键的点击来执行代码。例如使用up，down按键来移动一个方块，left，right按键来旋转一个元素，plus，minus按键来缩放一个元素。

```
// keys.qml

import QtQuick 2.0

DarkSquare {
    width: 400; height: 200

    GreenSquare {
        id: square
        x: 8; y: 8
    }
    focus: true
    Keys.onLeftPressed: square.x -= 8
    Keys.onRightPressed: square.x += 8
    Keys.onUpPressed: square.y -= 8
    Keys.onDownPressed: square.y += 8
    Keys.onPressed: {
        switch(event.key) {
            case Qt.Key_Plus:
                square.scale += 0.2
                break;
            case Qt.Key_Minus:
                square.scale -= 0.2
                break;
        }

    }
}
```

![](http://qmlbook.org/_images/keys.png)
