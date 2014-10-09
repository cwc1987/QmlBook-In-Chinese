# Qt5介绍（Qt5 Introduction）

## 1.2.1 Qt Quick

Qt Quick是Qt5中用户界面技术的涵盖。Qt Quick自身包含了以下几种技术：

* QML-使用于用户界面的标识语言

* JavaScript-动态脚本语言

* Qt C++-具有高度可移植性的C++库.

![](http://qmlbook.org/_images/qt5_overview.png)

类似HTML语言，QML是一个标识语言。它由QtQuick封装在Item {}的元素的标识组成。它从头设计了用户界面的创建，并且可以让开发人员快速，简单的理解。用户界面可以使用JavaScript代码来提供和加强更多的功能。Qt Quick可以使用你自己本地已有的Qt C++轻松快速的扩展它的能力。简单声明的UI被称作前端，本地部分被称作后端。这样你可以将程序的计算密集部分与来自应用程序用户界面操作部分分开。

在典型的项目中前端开发使用QML/JaveScript，后端代码开发使用Qt C++来完成系统接口和繁重的计算工作。这样就很自然的将设计界面的开发者和功能开发者分开了。后端开发测试使用Qt自有的单元测试框架后，导出给前端开发者使用。

## 1.2.2 一个用户界面（Digesting an User Interface）

让我们来使用QtQuick来创建一个简单的用户界面，展示QML语言某些方面的特性。最后我们将获得一个旋转的风车。

我们开始创建一个空的main.qml文档。所有的QML文件都已.qml作为后缀。作为一个标识语言（类似HTML）一个QML文档需要并且只有一个根元素，在我们的案例中是一个基于background的图像高度与宽度的几何图形元素：

```
import QtQuick 2.0

Image {
    id: root
    source: "images/background.png"
}
```

QML不会对根元素设置任何限制，我们使用一个backgournd图像作为资源的图像元素来作为我们的根元素。

![](http://qmlbook.org/_images/background.png)

**注意**

**每一个元素都有属性，比如一个图像有宽度，高度但是也有一些其它的属性例如资源。图像元素的大小能够自动的从图像大小上得出。否则我们应该设置宽度和高度属性来显示有效的像素。**

**大多数典型的元素都放置在QtQuick2.0模块中，我们首先应该在第一行作这个重要的声明。**

**id是这个特殊的属性是可选的，包含了一个标识符，在文档后面的地方可以直接引用。**

**重要提示：一个id属性无法在它被设置后改变，并且在程序执行期间无法被设置。使用root作为根元素id仅仅是作者的习惯，可以在比较大的QML文档中方便的引用最顶层元素。**

风车作为前景元素使用图像的方式放置在我们的用户界面上。

![](http://qmlbook.org/_images/pinwheel.png)

正常情况下你的用户界面应该有不同类型的元素构成，而不是像我们的例子一样只有图像元素。

```
Image {
    id: root
    ...
    Image {
        id: wheel
        anchors.centerIn: parent
        source: "images/pinwheel.png"
    }
    ...
}
```

为了把风车放在中间的位置，我们使用了一个复杂的属性，称之为锚。锚定允许你指定几何对象与父对象或者同级对象之间的位置关系。比如放置我在另一个元素中间（anchors.centerIn:parent）.有左边（left），右边（right），顶部（top），底部（bottom），中央（centerIn），填充（fill），垂直中央（verticalCenter）和水平中央（horizontalCenter）来表示元素之间的关系。确保他们能够匹配，锚定一个对象的左侧顶部的一个元素这样的做法是没有意义的。所以我们设置风车在父对象background的中央。

**注意**

**有时你需要进行一些微小的调整。使用anchors.horizontalCenterOffset或者anchors.verticalCenterOffset可以帮你实现这个功能。类似的调整属性也可以用于其他所有的锚。查阅Qt的帮助文档可以知道完整的锚属性列表。**

**注意**

**将一个图像作为根矩形元素的子元素放置展示了一种声明式语言的重要概念。你描述了用户界面的层和分组的顺序，最顶部的一层（根矩形框）先绘制，然后子层按照包含它的元素局部坐标绘制在包含它的元素上。**

为了让我们的展示更加有趣一点，我们应该让程序有一些交互功能。当用户点击场景上某个位置时，让我们的风车转动起来。

我们使用mouseArea元素，并且让它与我们的根元素大小一样。

```
Image {
    id: root
    ...
    MouseArea {
        anchors.fill: parent
        onClicked: wheel.rotation += 90
    }
    ...
}
```

当用户点击覆盖区域时，鼠标区域会发出一个信号。你可以重写onClicked函数来链接这个信号。在这个案例中引用了风车的图像并且让他旋转增加90度。

**注意**

**对于每个工作的信号，命名方式都是on + SignalName的标题。当属性的值发生改变时也会发出一个信号。它们的命名方式是：on + PropertyName + Chagned。
如果一个宽度（width）属性改变了，你可以使用onWidthChanged: print(width)来得到这个监控这个新的宽度值。**

现在风车将会旋转，但是还不够流畅。风车的旋转角度属性被直接改变了。我们应该怎样让90度的旋转可以持续一段时间呢。现在是动画效果发挥作用的时候了。一个动画定义了一个属性的在一段时间内的变化过程。为了实现这个效果，我们使用一个动画类型叫做属性行为。这个行为指定了一个动画来定义属性的每一次改变并赋值给属性。每次属性改变，动画都会运行。这是QML中声明动画的几种方式中的一种方式。

```
Image {
    id: root
    Image {
        id: wheel
        Behavior on rotation {
            NumberAnimation {
                duration: 250
            }
        }
    }
}
```

现在每当风车旋转角度发生改变时都会使用NumberAnimation来实现250毫秒的旋转动画效果。每一次90度的转变都需要花费250ms。

现在风车看起来好多了，我希望以上这些能够让你能够对Qt Quick编程有一些了解。
