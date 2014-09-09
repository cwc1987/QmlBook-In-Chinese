# 简单的转换（Simple Transformations）

转换操作改变了一个对象的几何状态。QML元素对象通常能够被平移，旋转，缩放。下面我们将讲解这些简单的操作和一些更高级的用法。
我们先从一个简单的转换开始。用下面的场景作为我们学习的开始。

简单的位移是通过改变x,y坐标来完成的。旋转是改变rotation（旋转）属性来完成的，这个值使用角度作为单位（0~360）。缩放是通过改变scale（比例）的属性来完成的，小于1意味着缩小，大于1意味着放大。旋转与缩放不会改变对象的几何形状，对象的x,y（坐标）与width/height（宽/高）也类似。只有绘制指令是被转换的对象。

在我们展示例子之前我想要介绍一些东西：ClickableImage元素（ClickableImage element），ClickableImage仅仅是一个包含鼠标区域的图像元素。我们遵循一个简单的原则，三次使用相同的代码描述一个用户界面最好可以抽象为一个组件。

```
// ClickableImage.qml

// Simple image which can be clicked

import QtQuick 2.0

Image {
    id: root
    signal clicked

    MouseArea {
        anchors.fill: parent
        onClicked: root.clicked()
    }
}
```

![](http://qmlbook.org/_images/rockets.png)

我们使用我们可点击图片元素来显示了三个火箭。当点击时，每个火箭执行一种简单的转换。点击背景将会重置场景。

```
// transformation.qml


import QtQuick 2.0

Item {
    // set width based on given background
    width: bg.width
    height: bg.height

    Image { // nice background image
        id: bg
        source: "assets/background.png"
    }

    MouseArea {
        id: backgroundClicker
        // needs to be before the images as order matters
        // otherwise this mousearea would be before the other elements
        // and consume the mouse events
        anchors.fill: parent
        onClicked: {
            // reset our little scene
            rocket1.x = 20
            rocket2.rotation = 0
            rocket3.rotation = 0
            rocket3.scale = 1.0
        }
    }

    ClickableImage {
        id: rocket1
        x: 20; y: 100
        source: "assets/rocket.png"
        onClicked: {
            // increase the x-position on click
            x += 5
        }
    }

    ClickableImage {
        id: rocket2
        x: 140; y: 100
        source: "assets/rocket.png"
        smooth: true // need antialising
        onClicked: {
            // increase the rotation on click
            rotation += 5
        }
    }

    ClickableImage {
        id: rocket3
        x: 240; y: 100
        source: "assets/rocket.png"
        smooth: true // need antialising
        onClicked: {
            // several transformations
            rotation += 5
            scale -= 0.05
        }
    }
}
```

![](http://qmlbook.org/_images/rockets_transformed.png)

火箭1在每次点击后X轴坐标增加5像素，火箭2每次点击后会旋转。火箭3每次点击后会缩小。对于缩放和旋转操作我们都设置了smooth:true来增加反锯齿，由于性能的原因通常是被关闭的（与剪裁属性clip类似）。当你看到你的图形中出现锯齿时，你可能就需要打开平滑（smooth）。

**注意**

**为了获得更好的显示效果，当缩放图片时推荐使用已缩放的图片来替代，过量的放大可能会导致图片模糊不清。当你在缩放图片时你最好考虑使用smooth:true来提高图片显示质量。**

使用MouseArea来覆盖整个背景，点击背景可以初始化火箭的值。

**注意**

**在代码中先出现的元素有更低的堆叠顺序（叫做z顺序值z-order），如果你点击火箭1足够多次，你会看见火箭1移动到了火箭2下面。z轴顺序也可以使用元素对象的z-property来控制。**

![](http://qmlbook.org/_images/order_matters.png)

**由于火箭2后出现在代码中，火箭2将会放在火箭1上面。这同样适用于MouseArea（鼠标区域），一个后出现在代码中的鼠标区域将会与之前的鼠标区域重叠，后出现的鼠标区域才能捕捉到鼠标事件。**

**请记住：文档中元素的顺序很重要。**
