# 定位元素（Positioning Element）

有一些QML元素被用于放置元素对象，它们被称作定位器，QtQuick模块提供了Row，Column，Grid，Flow用来作为定位器。你可以在下面的插图中看到它们使用相同内容的显示效果。

**注意**

**在我们详细介绍前，我们先介绍一些相关的元素，红色（red），蓝色（blue），绿色（green），高亮（lighter）与黑暗（darker）方块，每一个组件都包含了一个48乘48的着色区域。下面是关于RedSquare（红色方块）的代码：**

```
// RedSquare.qml

import QtQuick 2.0

Rectangle {
    width: 48
    height: 48
    color: "#ea7025"
    border.color: Qt.lighter(color)
}
```

**请注意使用了Qt.lighter（color）来指定了基于填充色的边界高亮色。我们将会在后面的例子中使用到这些元素，希望后面的代码能够容易读懂。请记住每一个矩形框的初始化大小都是48乘48像素大小。**

Column（列）元素将它的子对象通过顶部对齐的列方式进行排列。spacing属性用来设置每个元素之间的间隔大小。

![](http://qmlbook.org/_images/column.png)

```
// column.qml

import QtQuick 2.0

DarkSquare {
    id: root
    width: 120
    height: 240

    Column {
        id: column
        anchors.centerIn: parent
        spacing: 8
        RedSquare { }
        GreenSquare { width: 96 }
        BlueSquare { }
    }
}

// M1<<
```

Row（行）元素将它的子对象从左到右，或者从右到左依次排列，排列方式取决于layoutDirection属性。spacing属性用来设置每个元素之间的间隔大小。

![](http://qmlbook.org/_images/row.png)

```
// row.qml

import QtQuick 2.0

BrightSquare {
    id: root
    width: 400; height: 120

    Row {
        id: row
        anchors.centerIn: parent
        spacing: 20
        BlueSquare { }
        GreenSquare { }
        RedSquare { }
    }
}
```

Grid（栅格）元素通过设置rows（行数）和columns（列数）将子对象排列在一个栅格中。可以只限制行数或者列数。如果没有设置它们中的任意一个，栅格元素会自动计算子项目总数来获得配置，例如，设置rows（行数）为3，添加了6个子项目到元素中，那么会自动计算columns（列数）为2。属性flow（流）与layoutDirection（布局方向）用来控制子元素的加入顺序。spacing属性用来控制所有元素之间的间隔。

![](http://qmlbook.org/_images/grid.png)

```
// grid.qml

import QtQuick 2.0

BrightSquare {
    id: root
    width: 160
    height: 160

    Grid {
        id: grid
        rows: 2
        columns: 2
        anchors.centerIn: parent
        spacing: 8
        RedSquare { }
        RedSquare { }
        RedSquare { }
        RedSquare { }
    }

}
```

最后一个定位器是Flow（流）。通过flow（流）属性和layoutDirection（布局方向）属性来控制流的方向。它能够从头到底的横向布局，也可以从左到右或者从右到左进行布局。作为加入流中的子对象，它们在需要时可以被包装成新的行或者列。为了让一个流可以工作，必须指定一个宽度或者高度，可以通过属性直接设定，或者通过anchor（锚定）布局设置。

![](http://qmlbook.org/_images/flow.png)

```
// flow.qml

import QtQuick 2.0

BrightSquare {
    id: root
    width: 160
    height: 160

    Flow {
        anchors.fill: parent
        anchors.margins: 20
        spacing: 20
        RedSquare { }
        BlueSquare { }
        GreenSquare { }
    }
}
```

通常Repeater（重复元素）与定位器一起使用。它的工作方式就像for循环与迭代器的模式一样。在这个最简单的例子中，仅仅提供了一个循环的例子。

![](http://qmlbook.org/_images/repeater.png)

```
// repeater.qml

import QtQuick 2.0

DarkSquare {
    id: root
    width: 252
    height: 252
    property variant colorArray: ["#00bde3", "#67c111", "#ea7025"]


    Grid{
        anchors.fill: parent
        anchors.margins: 8
        spacing: 4
        Repeater {
            model: 16
            Rectangle {
                width: 56; height: 56
                property int colorIndex: Math.floor(Math.random()*3)
                color: root.colorArray[colorIndex]
                border.color: Qt.lighter(color)
                Text {
                    anchors.centerIn: parent
                    color: "#f0f0f0"
                    text: "Cell " + index
                }
            }
        }
    }
}
```

在这个重复元素的例子中，我们使用了一些新的方法。我们使用一个颜色数组定义了一组颜色属性。重复元素能够创建一连串的矩形框（16个，就像模型中定义的那样）。每一次的循环都会创建一个矩形框作为repeater的子对象。在矩形框中，我们使用了JS数学函数Math.floor(Math.random()*3)来选择颜色。这个函数会给我们生成一个0~2的随机数，我们使用这个数在我们的颜色数组中选择颜色。注意之前我们说过JavaScript是QtQuick中的一部分，所以这些典型的库函数我们都可以使用。

一个重复元素循环时有一个index（索引）属性值。当前的循环索引（0,1,2,....15）。我们可以使用这个索引值来做一些操作，例如在我们这个例子中使用Text（文本）显示当前索引值。

**注意**

**高级的大数据模型处理和使用动态代理的动态视图会在模型与视图（model-view）章节中讲解。当有一小部分的静态数据需要显示时，使用重复元素是最好的方式。**
