# 动画（Animations）

动画被用于属性的改变。一个动画定义了属性值改变的曲线，将一个属性值变化从一个值过渡到另一个值。动画是由一连串的目标属性活动定义的，平缓的曲线算法能够引发一个定义时间内属性的持续变化。所有在QtQuick中的动画都由同一个计时器来控制，因此它们始终都保持同步，这也提高了动画的性能和显示效果。

**注意**

**动画控制了属性的改变，也就是值的插入。这是一个基本的概念，QML是基于元素，属性与脚本的。每一个元素都提供了许多的属性，每一个属性都在等待使用动画。在这本书中你将会看到这是一个壮阔的场景，你会发现你自己在看一些动画时欣赏它们的美丽并且肯定自己的创造性想法。然后请记住：动画控制了属性的改变，每个元素都有大量的属性供你任意使用。**

![](http://qmlbook.org/_images/animation.png)

```
// animation.qml

import QtQuick 2.0

Image {
    source: "assets/background.png"

    Image {
        x: 40; y: 80
        source: "assets/rocket.png"

        NumberAnimation on x {
            to: 240
            duration: 4000
            loops: Animation.Infinite
        }
        RotationAnimation on rotation {
            to: 360
            duration: 4000
            loops: Animation.Infinite
        }
    }
}
```

上面这个例子在x坐标和旋转属性上应用了一个简单的动画。每一次动画持续4000毫秒并且永久循环。x轴坐标动画展示了火箭的x坐标逐渐移至240，旋转动画展示了当前角度到360度的旋转。两个动画同时运行，并且在加载用户界面完成后开始。

现在你可以通过to属性和duration属性来实现动画效果。或者你可以在opacity或者scale上添加动画作为例子，集成这两个参数，你可以实现火箭逐渐消失在太空中，试试吧!

## 5.1.1 动画元素（Animation Elements）

有几种类型的动画，每一种都在特定情况下都有最佳的效果，下面列出了一些常用的动画：

* PropertyAnimation（属性动画）- 使用属性值改变播放的动画

* NumberAnimation（数字动画）- 使用数字改变播放的动画

* ColorAnimation（颜色动画）- 使用颜色改变播放的动画

* RotationAnimation（旋转动画）- 使用旋转改变播放的动画

除了上面这些基本和通常使用的动画元素，QtQuick还提供了一切特殊场景下使用的动画：

* PauseAnimation（停止动画）- 运行暂停一个动画

* SequentialAnimation（顺序动画）- 允许动画有序播放

* ParallelAnimation（并行动画）- 允许动画同时播放

* AnchorAnimation（锚定动画）- 使用锚定改变播放的动画

* ParentAnimation（父元素动画）- 使用父对象改变播放的动画

* SmotthedAnimation（平滑动画）- 跟踪一个平滑值播放的动画

* SpringAnimation（弹簧动画）- 跟踪一个弹簧变换的值播放的动画

* PathAnimation（路径动画）- 跟踪一个元素对象的路径的动画

* Vector3dAnimation（3D容器动画）- 使用QVector3d值改变播放的动画

我们将在后面学习怎样创建一连串的动画。当使用更加复杂的动画时，我们可能需要在播放一个动画时中改变一个属性或者运行一个脚本。对于这个问题，QtQuick提供了一个动作元素：

* PropertyAction（属性动作）- 在播放动画时改变属性

* ScriptAction（脚本动作）- 在播放动画时运行脚本

在这一章中我们将会使用一些小的例子来讨论大多数类型的动画。

## 5.1.2 应用动画（Applying Animations）

动画可以通过以下几种方式来应用：

* 属性动画 - 在元素完整加载后自动运行

* 属性动作 - 当属性值改变时自动运行

* 独立运行动画 - 使用start()函数明确指定运行或者running属性被设置为true（比如通过属性绑定）

后面我们会谈论如何在状态变换时播放动画。

**扩展可点击图像元素版本2（ClickableImage Version2）**

为了演示动画的使用方法，我们重新实现了ClickableImage组件并且使用了一个文本元素（Text Element）来扩展它。

```
// ClickableImageV2.qml
// Simple image which can be clicked

import QtQuick 2.0

Item {
    id: root
    width: container.childrenRect.width
    height: container.childrenRect.height
    property alias text: label.text
    property alias source: image.source
    signal clicked

    Column {
        id: container
        Image {
            id: image
        }
        Text {
            id: label
            width: image.width
            horizontalAlignment: Text.AlignHCenter
            wrapMode: Text.WordWrap
            color: "#111111"
        }
    }

    MouseArea {
        anchors.fill: parent
        onClicked: root.clicked()
    }
}
```

为了给图片下面的元素定位，我们使用了Column（列）定位器，并且使用基于列的子矩形（childRect）属性来计算它的宽度和高度（width and height）。我们导出了文本（text）和图形源（source）属性，一个点击信号（clicked signal）。我们使用文本元素的wrapMode属性来设置文本与图像一样宽并且可以自动换行。

**注意**

**由于几何依赖关系的反向（父几何对象依赖于子几何对象）我们不能对ClickableImageV2设置宽度/高度（width/height），因为这样将会破坏我们已经做好的属性绑定。这是我们内部设计的限制，作为一个设计组件的人你需要明白这一点。通常我们更喜欢内部几何图像依赖于父几何对象。**

![](http://qmlbook.org/_images/animationtypes_start.png)

三个火箭位于相同的y轴坐标（y = 200）。它们都需要移动到y = 40。每一个火箭都使用了一种的方法来完成这个功能。

```
    ClickableImageV3 {
        id: rocket1
        x: 40; y: 200
        source: "assets/rocket2.png"
        text: "animation on property"
        NumberAnimation on y {
            to: 40; duration: 4000
        }
    }
```

**第一个火箭**


第一个火箭使用了Animation on <property>属性变化的策略来完成。动画会在加载完成后立即播放。点击火箭可以重置它回到开始的位置。在动画播放时重置第一个火箭不会有任何影响。在动画开始前的几分之一秒设置一个新的y轴坐标让人感觉挺不安全的，应当避免这样的属性值竞争的变化。

```
    ClickableImageV3 {
        id: rocket2
        x: 152; y: 200
        source: "assets/rocket2.png"
        text: "behavior on property"
        Behavior on y {
            NumberAnimation { duration: 4000 }
        }

        onClicked: y = 40
        // random y on each click
    //        onClicked: y = 40+Math.random()*(205-40)
    }
```

**第二个火箭**

第二个火箭使用了behavior on <property>属性行为策略的动画。这个行为告诉属性值每时每刻都在变化，通过动画的方式来改变这个值。可以使用行为元素的enabled : false来设置行为失效。当你点击这个火箭时它将会开始运行（y轴坐标逐渐移至40）。然后其它的点击对于位置的改变没有任何的影响。你可以试着使用一个随机值（例如 40+(Math.random()*(205-40)）来设置y轴坐标。你可以发现动画始终会将移动到新位置的时间匹配在4秒内完成。

```
    ClickableImageV3 {
        id: rocket3
        x: 264; y: 200
        source: "assets/rocket2.png"
        onClicked: anim.start()
    //        onClicked: anim.restart()

        text: "standalone animation"

        NumberAnimation {
            id: anim
            target: rocket3
            properties: "y"
            from: 205
            to: 40
            duration: 4000
        }
    }
```

**第三个火箭**

第三个火箭使用standalone animation独立动画策略。这个动画由一个私有的元素定义并且可以写在文档的任何地方。点击火箭调用动画函数start()来启动动画。每一个动画都有start()，stop()，resume()，restart()函数。这个动画自身可以比其他类型的动画更早的获取到更多的相关信息。我们只需要定义目标和目标元素的属性需要怎样改变的一个动画。我们定义一个to属性的值，在这个例子中我们也定义了一个from属性的值允许动画可以重复运行。

![](http://qmlbook.org/_images/animationtypes.png)

点击背景能够重新设置所有的火箭回到它们的初始位置。第一个火箭无法被重置，只有重启程序重新加载元素才能重置它。

**注意**

**另一个启动/停止一个动画的方法是绑定一个动画的running属性。当需要用户输入控制属性时这种方法非常有用：**

```
    NumberAnimation {
        ...
        // animation runs when mouse is pressed
        running: area.pressed
    }
    MouseArea {
        id: area
    }
```

## 5.1.3 缓冲曲线（Easing Curves）

属性值的改变能够通过一个动画来控制，缓冲曲线属性影响了一个属性值改变的插值算法。我们现在已经定义的动画都使用了一种线性的插值算法，因为一个动画的默认缓冲类型是Easing.Linear。在一个小场景下的x轴与y轴坐标改变可以得到最好的视觉效果。一个线性插值算法将会在动画开始时使用from的值到动画结束时使用的to值绘制一条直线，所以缓冲类型定义了曲线的变化情况。精心为一个移动的对象挑选一个合适的缓冲类型将会使界面更加自然，例如一个页面的滑出，最初使用缓慢的速度滑出，然后在最后滑出时使用高速滑出，类似翻书一样的效果。

**注意**

**不要过度的使用动画。用户界面动画的设计应该尽量小心，动画是让界面更加生动而不是充满整个界面。眼睛对于移动的东西非常敏感，很容易干扰用户的使用。**

在下面的例子中我们将会使用不同的缓冲曲线，每一种缓冲曲线都都使用了一个可点击图片来展示，点击将会在动画中设置一个新的缓冲类型并且使用这种曲线重新启动动画。

![](http://qmlbook.org/_images/easingtypes.png)

**扩展可点击图像V3（ClickableImage V3）**

我们给图片和文本添加了一个小的外框来增强我们的ClickableImage。添加一个属性property bool framed: false来作为我们的API，基于framed的值我们能够设置这个框是否可见，并且不破坏之前用户的使用。下面是我们做的修改。

```
// ClickableImageV2.qml
// Simple image which can be clicked

import QtQuick 2.0

Item {
    id: root
    width: container.childrenRect.width + 16
    height: container.childrenRect.height + 16
    property alias text: label.text
    property alias source: image.source
    signal clicked

    // M1>>
    // ... add a framed rectangle as container
    property bool framed : false

    Rectangle {
        anchors.fill: parent
        color: "white"
        visible: root.framed
    }
```

这个例子的代码非常简洁。我们使用了一连串的缓冲曲线的名称（property variant easings）并且在一个Repeater（重复元素）中将它们分配给一个ClickableImage。图片的源路径通过一个命名方案来定义，一个叫做“InQuad”的缓冲曲线在“curves/InQuad.png”中有一个对应的图片。如果你点击一个曲线图，这个点击将会分配一个缓冲类型给动画然后重新启动动画。动画自身是用来设置方块的x坐标属性在2秒内变化的独立动画。

```
// easingtypes.qml

import QtQuick 2.0

DarkSquare {
    id: root
    width: 600
    height: 340

    // A list of easing types
    property variant easings : [
        "Linear", "InQuad", "OutQuad", "InOutQuad",
        "InCubic", "InSine", "InCirc", "InElastic",
        "InBack", "InBounce" ]


    Grid {
        id: container
        anchors.top: parent.top
        anchors.horizontalCenter: parent.horizontalCenter
        anchors.margins: 16
        height: 200
        columns: 5
        spacing: 16
        // iterates over the 'easings' list
        Repeater {
            model: easings
            ClickableImageV3 {
                framed: true
                // the current data entry from 'easings' list
                text: modelData
                source: "curves/" + modelData + ".png"
                onClicked: {
                    // set the easing type on the animation
                    anim.easing.type = modelData
                    // restart the animation
                    anim.restart()
                }
            }
        }
    }

    // The square to be animated
    GreenSquare {
        id: square
        x: 40; y: 260
    }

    // The animation to test the easing types
    NumberAnimation {
        id: anim
        target: square
        from: 40; to: root.width - 40 - square.width
        properties: "x"
        duration: 2000
    }
}
```

当你运行这个例子时，请注意观察动画的改变速度。一些动画对于这个对象看起来很自然，一些看起来非常恼火。

除了duration属性与easing.type属性，你也可以对动画进行微调。例如PropertyAnimation属性，大多数动画都支持附加的easing.amplitude（缓冲振幅），easing.overshoot（缓冲溢出），easing.period（缓冲周期），这些属性允许你对个别的缓冲曲线进行微调。不是所有的缓冲曲线都支持这些参数。可以查看Qt PropertyAnimation文档中的缓冲列表（easing table）来查看一个缓冲曲线的相关参数。

**注意**

**对于用户界面正确的动画非常重要。请记住动画是帮助用户界面更加生动而不是刺激用户的眼睛。**

## 5.1.4 动画分组（Grouped Animations）

通常使用的动画比一个属性的动画更加复杂。例如你想同时运行几个动画并把他们连接起来，或者在一个一个的运行，或者在两个动画之间执行一个脚本。动画分组提供了很好的帮助，作为命名建议可以叫做一组动画。有两种方法来分组：平行与连续。你可以使用SequentialAnimation（连续动画）和ParallelAnimation（平行动画）来实现它们，它们作为动画的容器来包含其它的动画元素。

![](http://qmlbook.org/_images/groupedanimation.png)

当开始时，平行元素的所有子动画都会平行运行，它允许你在同一时间使用不同的属性来播放动画。

```
// parallelanimation.qml
import QtQuick 2.0

BrightSquare {
    id: root
    width: 300
    height: 200
    property int duration: 3000

    ClickableImageV3 {
        id: rocket
        x: 20; y: 120
        source: "assets/rocket2.png"
        onClicked: anim.restart()
    }

    ParallelAnimation {
        id: anim
        NumberAnimation {
            target: rocket
            properties: "y"
            to: 20
            duration: root.duration
        }
        NumberAnimation {
            target: rocket
            properties: "x"
            to: 160
            duration: root.duration
        }
    }
}
```

![](http://qmlbook.org/_images/parallelanimation_sequence.png)

一个连续的动画将会一个一个的运行子动画。

```
// sequentialanimation.qml
import QtQuick 2.0

BrightSquare {
    id: root
    width: 300
    height: 200
    property int duration: 3000

    ClickableImageV3 {
        id: rocket
        x: 20; y: 120
        source: "assets/rocket2.png"
        onClicked: anim.restart()
    }

    SequentialAnimation {
        id: anim
        NumberAnimation {
            target: rocket
            properties: "y"
            to: 20
            // 60% of time to travel up
            duration: root.duration*0.6
        }
        NumberAnimation {
            target: rocket
            properties: "x"
            to: 160
            // 40% of time to travel sideways
            duration: root.duration*0.4
        }
    }
}
```

![](http://qmlbook.org/_images/sequentialanimation_sequence.png)

分组动画也可以被嵌套，例如一个连续动画可以拥有两个平行动画作为子动画。我们来看看这个足球的例子。这个动画描述了一个从左向右扔一个球的行为：

![](http://qmlbook.org/_images/soccer_init.png)

要弄明白这个动画我们需要剖析这个目标的运动过程。我们需要记住这个动画是通过属性变化来实现的动画，下面是不同部分的转换：

* 从左向右的x坐标转换（X1）。

* 从下往上的y坐标转换（Y1）然后跟着一个从上往下的Y坐标转换（Y2）。

* 整个动画过程中360度旋转。

这个动画将会花掉3秒钟的时间。

![](http://qmlbook.org/_images/soccer_plan.png)

我们使用一个空的基本元素对象（Item）作为根元素，它的宽度为480，高度为300。

```
import QtQuick 1.1

Item {
    id: root
    width: 480
    height: 300
    property int duration: 3000

    ...
}
```

我们定义动画的总持续时间作为参考，以便更好的同步各部分的动画。

下一步我们需需要添加一个背景，在我们这个例子中有两个矩形框分别使用了绿色渐变和蓝色渐变填充。

```
    Rectangle {
        id: sky
        width: parent.width
        height: 200
        gradient: Gradient {
            GradientStop { position: 0.0; color: "#0080FF" }
            GradientStop { position: 1.0; color: "#66CCFF" }
        }
    }
    Rectangle {
        id: ground
        anchors.top: sky.bottom
        anchors.bottom: root.bottom
        width: parent.width
        gradient: Gradient {
            GradientStop { position: 0.0; color: "#00FF00" }
            GradientStop { position: 1.0; color: "#00803F" }
        }
    }
```

![](http://qmlbook.org/_images/soccer_stage1.png)

上面部分的蓝色区域高度为200像素，下面部分的区域使用上面的蓝色区域的底作为锚定的顶，使用根元素的底作为底。

让我们将足球加入到屏幕上，足球是一个图片，位于路径“assets/soccer_ball.png”。首先我们需要将它放置在左下角接近边界处。

```
    Image {
        id: ball
        x: 20; y: 240
        source: "assets/soccer_ball.png"

        MouseArea {
            anchors.fill: parent
            onClicked: {
                ball.x = 20; ball.y = 240
                anim.restart()
            }
        }
    }
```

![](http://qmlbook.org/_images/soccer_stage2.png)

图片与鼠标区域连接，点击球将会重置球的状态，并且动画重新开始。

首先使用一个连续的动画来播放两次的y轴变换。

```
    SequentialAnimation {
        id: anim
        NumberAnimation {
            target: ball
            properties: "y"
            to: 20
            duration: root.duration * 0.4
        }
        NumberAnimation {
            target: ball
            properties: "y"
            to: 240
            duration: root.duration * 0.6
        }
    }
```

![](http://qmlbook.org/_images/soccer_stage3.png)

在动画总时间的40%的时间里完成上升部分，在动画总时间的60%的时间里完成下降部分，一个动画完成后播放下一个动画。目前还没有使用任何缓冲曲线。缓冲曲线将在后面使用easing curves来添加，现在我们只关心如何使用动画来完成过渡。

现在我们需要添加x轴坐标转换。x轴坐标转换需要与y轴坐标转换同时进行，所以我们需要将y轴坐标转换的连续动画和x轴坐标转换一起压缩进一个平行动画中。

```
    ParallelAnimation {
        id: anim
        SequentialAnimation {
            // ... our Y1, Y2 animation
        }
        NumberAnimation { // X1 animation
            target: ball
            properties: "x"
            to: 400
            duration: root.duration
        }
    }
```

![](http://qmlbook.org/_images/soccer_stage4.png)

最后我们想要旋转这个球，我们需要向平行动画中添加一个新的动画，我们选择RotationAnimation来实现旋转。

```
    ParallelAnimation {
        id: anim
        SequentialAnimation {
            // ... our Y1, Y2 animation
        }
        NumberAnimation { // X1 animation
            // X1 animation
        }
        RotationAnimation {
            target: ball
            properties: "rotation"
            to: 720
            duration: root.duration
        }
    }
```

我们已经完成了整个动画链表，然后我们需要给动画提供一个正确的缓冲曲线来描述一个移动的球。对于Y1动画我们使用Easing.OutCirc缓冲曲线，它看起来更像是一个圆周运动。Y2使用了Easing.OutBounce缓冲曲线，因为在最后球会发生反弹。（试试使用Easing.InBounce，你会发现反弹将会立刻开始。）。X1和ROT1动画都使用线性曲线。

下面是这个动画最后的代码，提供给你作为参考：

```
    ParallelAnimation {
        id: anim
        SequentialAnimation {
            NumberAnimation {
                target: ball
                properties: "y"
                to: 20
                duration: root.duration * 0.4
                easing.type: Easing.OutCirc
            }
            NumberAnimation {
                target: ball
                properties: "y"
                to: 240
                duration: root.duration * 0.6
                easing.type: Easing.OutBounce
            }
        }
        NumberAnimation {
            target: ball
            properties: "x"
            to: 400
            duration: root.duration
        }
        RotationAnimation {
            target: ball
            properties: "rotation"
            to: 720
            duration: root.duration * 1.1
        }
    }
```
