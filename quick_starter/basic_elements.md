# 基本元素（Basic Elements）

元素可以被分为可视化元素与非可视化元素。一个可视化元素（例如矩形框Rectangle）有着几何形状并且可以在屏幕上显示。一个非可视化元素（例如计时器Timer）提供了常用的功能，通常用于操作可视化元素。

现在我们将专注于几个基础的可视化元素，例如Item（基础元素对象），Rectangle（矩形框），Text（文本），Image（图像）和MouseArea（鼠标区域）。

## 4.2.1 基础元素对象（Item Element）

Item（基础元素对象）是所有可视化元素的基础对象，所有其它的可视化元素都继承自Item。它自身不会有任何绘制操作，但是定义了所有可视化元素共有的属性：

| Group（分组） | 	Properties（属性） |
| -- | -- |
| Geometry（几何属性） | x,y（坐标）定义了元素左上角的位置，width，height（长和宽）定义元素的显示范围，z（堆叠次序）定义元素之间的重叠顺序。 |
| Layout handling（布局操作）| anchors（锚定），包括左（left），右（right），上（top），下（bottom），水平与垂直居中（vertical center，horizontal center），与margins（间距）一起定义了元素与其它元素之间的位置关系。 |
| Key handlikng（按键操作） | 附加属性key（按键）和keyNavigation（按键定位）属性来控制按键操作，处理输入焦点（focus）可用操作。 |
| Transformation（转换） | 缩放（scale）和rotate（旋转）转换，通用的x,y,z属性列表转换（transform），旋转基点设置（transformOrigin）。 |
| Visual（可视化） | 不透明度（opacity）控制透明度，visible（是否可见）控制元素是否显示，clip（裁剪）用来限制元素边界的绘制，smooth（平滑）用来提高渲染质量。 |
| State definition（状态定义） | states（状态列表属性）提供了元素当前所支持的状态列表，当前属性的改变也可以使用transitions（转变）属性列表来定义状态转变动画。 |

为了更好的理解不同的属性，我们将会在这章中尽量的介绍这些元素的显示效果。请记住这些基本的属性在所有可视化元素中都是可以使用的，并且在这些元素中的工作方式都是相同的。

**注意**

**Item（基本元素对象）通常被用来作为其它元素的容器使用，类似HTML语言中的div元素（div element）。**

## 4.2.2 矩形框元素（Rectangle Element）

Rectangle（矩形框）是基本元素对象的一个扩展，增加了一个颜色来填充它。它还支持边界的定义，使用border.color（边界颜色），border.width（边界宽度）来自定义边界。你可以使用radius（半径）属性来创建一个圆角矩形。

```
    Rectangle {
        id: rect1
        x: 12; y: 12
        width: 76; height: 96
        color: "lightsteelblue"
    }
    Rectangle {
        id: rect2
        x: 112; y: 12
        width: 76; height: 96
        border.color: "lightsteelblue"
        border.width: 4
        radius: 8
    }
```

![](http://qmlbook.org/_images/rectangle2.png)

**注意**

**颜色的命名是来自SVG颜色的名称（查看[http://www.w3.org/TR/css3-color/#svg-color]( http://www.w3.org/TR/css3-color/#svg-color)可以获取更多的颜色名称）。你也可以使用其它的方法来指定颜色，比如RGB字符串（'#FF4444'），或者一个颜色名字（例如'white'）。**

此外，填充的颜色与矩形的边框也支持自定义的渐变色。

```
    Rectangle {
        id: rect1
        x: 12; y: 12
        width: 176; height: 96
        gradient: Gradient {
            GradientStop { position: 0.0; color: "lightsteelblue" }
            GradientStop { position: 1.0; color: "slategray" }
        }
        border.color: "slategray"
    }
```

![](http://qmlbook.org/_images/rectangle3.png)

一个渐变色是由一系列的梯度值定义的。每一个值定义了一个位置与颜色。位置标记了y轴上的位置（0 = 顶，1 = 底）。GradientStop（倾斜点）的颜色标记了颜色的位置。

**注意**

**一个矩形框如果没有width/height（宽度与高度）将不可见。如果你有几个相互关联width/height（宽度与高度）的矩形框，在你组合逻辑中出了错后可能就会发生矩形框不可见，请注意这一点。**

**注意**

**这个函数无法创建一个梯形，最好使用一个已有的图像来创建梯形。有一种可能是在旋转梯形时，旋转的矩形几何结构不会发生改变，但是这会导致几何元素相同的可见区域的混淆。从作者的观点来看类似的情况下最好使用设计好的梯形图形来完成绘制。**

## 4.2.3 文本元素（Text Element）

显示文本你需要使用Text元素（Text Element）。它最值得注意的属性时字符串类型的text属性。这个元素会使用给出的text（文本）与font（字体）来计算初始化的宽度与高度。可以使用字体属性组来（font property group）来改变当前的字体，例如font.family，font.pixelSize，等等。改变文本的颜色值只需要改变颜色属性就可以了。

```
    Text {
        text: "The quick brown fox"
        color: "#303030"
        font.family: "Ubuntu"
        font.pixelSize: 28
    }
```

![](http://qmlbook.org/_images/text.png)

文本可以使用horizontalAlignment与verticalAlignment属性来设置它的对齐效果。为了提高文本的渲染效果，你可以使用style和styleColor属性来配置文字的外框效果，浮雕效果或者凹陷效果。对于过长的文本，你可能需要使用省略号来表示，例如A very ... long text，你可以使用elide属性来完成这个操作。elide属性允许你设置文本左边，右边或者中间的省略位置。如果你不想'....'省略号出现，并且希望使用文字换行的方式显示所有的文本，你可以使用wrapMode属性（这个属性只在明确设置了宽度后才生效）：

```
Text {
    width: 40; height: 120
    text: 'A very long text'
    // '...' shall appear in the middle
    elide: Text.ElideMiddle
    // red sunken text styling
    style: Text.Sunken
    styleColor: '#FF4444'
    // align text to the top
    verticalAlignment: Text.AlignTop
    // only sensible when no elide mode
    // wrapMode: Text.WordWrap
}
```

一个text元素（text element）只显示的文本，它不会渲染任何背景修饰。除了显示的文本，text元素背景是透明的。为一个文本元素提供背景是你自己需要考虑的问题。

**注意**

**知道一个文本元素（Text Element）的初始宽度与高度是依赖于文本字符串和设置的字体这一点很重要。一个没有设置宽度或者文本的文本元素（Text Element）将不可见，默认的初始宽度是0。**

**注意**

**通常你想要对文本元素布局时，你需要区分文本在文本元素内部的边界对齐和由元素边界自动对齐。前一种情况你需要使用horizontalAlignment和verticalAlignment属性来完成，后一种情况你需要操作元素的几何形状或者使用anchors（锚定）来完成。**

## 4.2.4 图像元素（Image Element）


一个图像元素（Image Element）能够显示不同格式的图像（例如PNG,JPG,GIF,BMP）。想要知道更加详细的图像格式支持信息，可以查看Qt的相关文档。source属性（source property）提供了图像文件的链接信息，fillMode（文件模式）属性能够控制元素对象的大小调整行为。

```
    Image {
        x: 12; y: 12
        // width: 48
        // height: 118
        source: "assets/rocket.png"
    }
    Image {
        x: 112; y: 12
        width: 48
        height: 118/2
        source: "assets/rocket.png"
        fillMode: Image.PreserveAspectCrop
        clip: true
    }
```

![](http://qmlbook.org/_images/image.png)

**注意**

**一个URL可以是使用'/'语法的本地路径（"./images/home.png"）或者一个网络链接（"[http://example.org/home.png](http://example.org/home.png)"）。**

**注意**

**图像元素（Image element）使用PreserveAspectCrop可以避免裁剪图像数据被渲染到图像边界外。默认情况下裁剪是被禁用的（clip:false）。你需要打开裁剪（clip:true）来约束边界矩形的绘制。这对任何可视化元素都是有效的。**

**建议**

**使用QQmlImageProvider你可以通过C++代码来创建自己的图像提供器，这允许你动态创建图像并且使用线程加载。**

## 4.2.5 鼠标区域元素（MouseArea Element）


为了与不同的元素交互，你通常需要使用MouseArea（鼠标区域）元素。这是一个矩形的非可视化元素对象，你可以通过它来捕捉鼠标事件。当用户与可视化端口交互时，mouseArea通常被用来与可视化元素对象一起执行命令。

```
    Rectangle {
        id: rect1
        x: 12; y: 12
        width: 76; height: 96
        color: "lightsteelblue"
        MouseArea {
            id: area
            width: parent.width
            height: parent.height
            onClicked: rect2.visible = !rect2.visible
        }
    }

    Rectangle {
        id: rect2
        x: 112; y: 12
        width: 76; height: 96
        border.color: "lightsteelblue"
        border.width: 4
        radius: 8
    }
```

![](http://qmlbook.org/_images/mousearea1.png)

![](http://qmlbook.org/_images/mousearea2.png)

**注意**

**这是QtQuick中非常重要的概念，输入处理与可视化显示分开。这样你的交互区域可以比你显示的区域大很多。**
