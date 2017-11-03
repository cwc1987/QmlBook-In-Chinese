# 1.2 Qt5介绍

## 1.2.1 Qt Quick

Qt Quick是Qt5界面开发技术的统称，是以下几种技术的集合：

* QML - 界面标记语言

* JavaScript - 动态脚本语言

* Qt C++ - 跨平台C++封装库

![](http://qmlbook.org/_images/qt5_overview.png)

QML是与HTML类似的一种标记语言。在QtQuick中将由标签组成的元素封装在大括号中`Item{}`。这样的设计重新定义了界面的创建方式，对于开发者而言更加简单易读。可以使用JavaScript开发界面功能，也可以使用本地Qt C++函数接口扩展界面功能。简单来说，声明式的UI被称作前端，本地C++部分称作后端，将复杂的计算过程与本地设备操作从界面开发中分离。

在一个典型的Qt5项目中，前端采用QML/JaveScript开发界面，后端采用Qt C++与系统交互并完成复杂的运算逻辑，将侧重设计的界面开发与功能开发的工作内容分离。通常后端开发者可以使用Qt的单元测试框架完成单元测试后将函数接口提供给前端开发者使用。

## 1.2.2 Qt5用户界面开发示例

我们将使用QtQuick创建一个简单的界面，这个例子展示了QML语言的一些特性，在例子完成后我们将获得一个可以旋转的风车。![](http://qmlbook.github.io/_images/scene.png)

我们开始创建一个空的`main.qml`文档。QML文件采用`.qml`作为文件格式后缀。作为一种标记语言（类似HTML）一个QML文档有且只有一个根元素，在这个例子中使用`Image`元素作为根元素，这个元素的宽度、高度与`"images/background.png"`图像相同。

```QML
import QtQuick 2.5

Image {
    id: root
    source: "images/background.png"
}
```

QML中不限制根元素类型，在上面这段代码中我们设置了`Image`元素的`source`属性作为我们的背景图像，它也是我们的根元素。

![](http://qmlbook.org/_images/background.png)

**注意**

**每个元素都有属性，比如**`Image`**有**`width`**和**`height`**，也会有其它的属性如**`source`**。**`Image`**元素的尺寸会自动与**`source`**设置的图像匹配。想要自定义**`Image`**元素的尺寸必须显式的定义**`width`**和**`height`**的值。**

**大多数标准元素都在**`QtQuick`**模块中，通常我们在导入声明中首先包含这个模块。**

`id`**是个特殊的属性，它作为一个标识符可以在其它地方引用元素，需要注意只对当前QML文档有效。**

---herer-------

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
如果一个宽度（width）属性改变了，你可以使用onWidthChanged: print\(width\)来得到这个监控这个新的宽度值。**

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

