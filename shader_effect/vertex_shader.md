# 顶点着色器（Vertex Shader）

顶点着色器用来操作ShaderEffect提供的顶点。正常情况下，ShaderEffect有4个顶点（左上top-left，右上top-right，左下bottom-left，右下bottom-right）。每个顶点使用vec4类型记录。为了实现顶点着色器的可视化，我们将编写一个吸收的效果。这个效果通常被用来让一个矩形窗口消失为一个点。

![](http://qmlbook.org/_images/genieeffect.png)

**配置场景（Setting up the scene）**

首先我们再一次配置场景。

```
import QtQuick 2.0

Rectangle {
    width: 480; height: 240
    color: '#1e1e1e'

    Image {
        id: sourceImage
        width: 160; height: width
        source: "assets/lighthouse.jpg"
        visible: false
    }
    Rectangle {
        width: 160; height: width
        anchors.centerIn: parent
        color: '#333333'
    }
    ShaderEffect {
        id: genieEffect
        width: 160; height: width
        anchors.centerIn: parent
        property variant source: sourceImage
        property bool minimized: false
        MouseArea {
            anchors.fill: parent
            onClicked: genieEffect.minimized = !genieEffect.minimized
        }
    }
}
```

这个场景使用了一个黑色背景，并且提供了一个使用图片作为资源纹理的ShaderEffect。使用image元素的原图片是不可见的，只是给我们的吸收效果提供资源。此外我们在ShaderEffect的位置添加了一个同样大小的黑色矩形框，这样我们可以更加明确的知道我们需要点击哪里来重置效果。

![](http://qmlbook.org/_images/geniescene.png)

点击图片将会触发效果，MouseArea覆盖了ShaderEffect。在onClicked操作中，我们绑定了自定义的布尔变量属性minimized。我们稍后使用这个属性来触发效果。

**最小化与正常化（Minimize and normalize）**

在我们配置好场景后，我们定义一个real类型的属性，叫做minimize，这个属性包含了我们当前最小化的值。这个值在0.0到1.0之间，由一个连续的动画来控制它。

```
        property real minimize: 0.0

        SequentialAnimation on minimize {
            id: animMinimize
            running: genieEffect.minimized
            PauseAnimation { duration: 300 }
            NumberAnimation { to: 1; duration: 700; easing.type: Easing.InOutSine }
            PauseAnimation { duration: 1000 }
        }

        SequentialAnimation on minimize {
            id: animNormalize
            running: !genieEffect.minimized
            NumberAnimation { to: 0; duration: 700; easing.type: Easing.InOutSine }
            PauseAnimation { duration: 1300 }
        }
```

这个动画绑定了由minimized属性触发。现在我们已经配置好我们的环境，最后让我们看看顶点着色器的代码。

```
        vertexShader: "
            uniform highp mat4 qt_Matrix;
            attribute highp vec4 qt_Vertex;
            attribute highp vec2 qt_MultiTexCoord0;
            varying highp vec2 qt_TexCoord0;
            uniform highp float minimize;
            uniform highp float width;
            uniform highp float height;
            void main() {
                qt_TexCoord0 = qt_MultiTexCoord0;
                highp vec4 pos = qt_Vertex;
                pos.y = mix(qt_Vertex.y, height, minimize);
                pos.x = mix(qt_Vertex.x, width, minimize);
                gl_Position = qt_Matrix * pos;
            }"
```

顶点着色器被每个顶点调用，在我们这个例子中，一共调用了四次。默认下提供qt已定义的参数，如qt_Matrix，qt_Vertex，qt_MultiTexCoord0，qt_TexCoord0。我们在之前已经讨论过这些变量。此外我们从ShaderEffect中链接minimize，width与height的值到我们的顶点着色器代码中。在main函数中，我们将当前纹理值保存在qt_TexCoord()中，让它在片段着色器中可用。现在我们拷贝当前位置，并修改顶点的x,y的位置。

```
highp vec4 pos = qt_Vertex;
pos.y = mix(qt_Vertex.y, height, minimize);
pos.x = mix(qt_Vertex.x, width, minimize);
```

mix(...)函数提供了一种在两个参数之间（0.0到1.0）的线性插值的算法。在我们的例子中，在当前y值与高度值之间基于minimize的值插值获得y值，x的值获取类似。记住minimize的值是由我们的连续动画控制，并且在0.0到1.0之间（反之亦然）。

![](http://qmlbook.org/_images/genieminimize.png)

这个结果的效果不是真正吸收效果，但是已经能朝着这个目标完成了一大步。

**基础弯曲（Primitive Bending）**

我们已经完成了最小化我们的坐标。现在我们想要修改一下对x值的操作，让它依赖当前的y值。这个改变很简单。y值计算在前。x值的插值基于当前顶点的y坐标。

```
highp float t = pos.y / height;
pos.x = mix(qt_Vertex.x, width, t * minimize);
```

这个结果造成当y值比较大时，x的位置更靠近width的值。也就是说上面2个顶点根本不受影响，它们的y值始终为0，下面两个顶点的x坐标值更靠近width的值，它们最后转向同一个x值。

![](http://qmlbook.org/_images/geniebending.png)

```
import QtQuick 2.0

Rectangle {
    width: 480; height: 240
    color: '#1e1e1e'

    Image {
        id: sourceImage
        width: 160; height: width
        source: "assets/lighthouse.jpg"
        visible: false
    }
    Rectangle {
        width: 160; height: width
        anchors.centerIn: parent
        color: '#333333'
    }
    ShaderEffect {
        id: genieEffect
        width: 160; height: width
        anchors.centerIn: parent
        property variant source: sourceImage
        property real minimize: 0.0
        property bool minimized: false


        SequentialAnimation on minimize {
            id: animMinimize
            running: genieEffect.minimized
            PauseAnimation { duration: 300 }
            NumberAnimation { to: 1; duration: 700; easing.type: Easing.InOutSine }
            PauseAnimation { duration: 1000 }
        }

        SequentialAnimation on minimize {
            id: animNormalize
            running: !genieEffect.minimized
            NumberAnimation { to: 0; duration: 700; easing.type: Easing.InOutSine }
            PauseAnimation { duration: 1300 }
        }


        vertexShader: "
            uniform highp mat4 qt_Matrix;
            uniform highp float minimize;
            uniform highp float height;
            uniform highp float width;
            attribute highp vec4 qt_Vertex;
            attribute highp vec2 qt_MultiTexCoord0;
            varying highp vec2 qt_TexCoord0;
            void main() {
                qt_TexCoord0 = qt_MultiTexCoord0;
                // M1>>
                highp vec4 pos = qt_Vertex;
                pos.y = mix(qt_Vertex.y, height, minimize);
                highp float t = pos.y / height;
                pos.x = mix(qt_Vertex.x, width, t * minimize);
                gl_Position = qt_Matrix * pos;
```

**更好的弯曲（Better Bending）**

现在简单的弯曲并不能真正的满足我们的要求，我们将添加几个部件来提升它的效果。首先我们增加动画，支持一个自定义的弯曲属性。这是非常必要的，由于弯曲立即发生，y值的最小化需要被推迟。两个动画在同一持续时间计算总和（300+700+100与700+1300）。

```
        property real bend: 0.0
        property bool minimized: false


        // change to parallel animation
        ParallelAnimation {
            id: animMinimize
            running: genieEffect.minimized
            SequentialAnimation {
                PauseAnimation { duration: 300 }
                NumberAnimation {
                    target: genieEffect; property: 'minimize';
                    to: 1; duration: 700;
                    easing.type: Easing.InOutSine
                }
                PauseAnimation { duration: 1000 }
            }
            // adding bend animation
            SequentialAnimation {
                NumberAnimation {
                    target: genieEffect; property: 'bend'
                    to: 1; duration: 700;
                    easing.type: Easing.InOutSine }
                PauseAnimation { duration: 1300 }
            }
        }
```

此外，为了使弯曲更加平滑，不再使用y值影响x值的弯曲函数，pos.x现在依赖新的弯曲属性动画：

```
highp float t = pos.y / height;
t = (3.0 - 2.0 * t) * t * t;
pos.x = mix(qt_Vertex.x, width, t * bend);
```

弯曲从0.0平滑开始，逐渐加快，在1.0时逐渐平滑。下面是这个函数在指定范围内的曲线图。对于我们，只需要关注0到1的区间。

![](http://qmlbook.org/_images/curve.png)

想要获得最大化的视觉改变，需要增加我们的顶点数量。可以使用网眼（mesh）来增加顶点：

```
mesh: GridMesh { resolution: Qt.size(16, 16) }
```

现在ShaderEffect被分布为16x16顶点的网格，替换了之前2x2的顶点。这样顶点之间的插值将会看起来更加平滑。

![](http://qmlbook.org/_images/geniesmoothbending.png)

你可以看见曲线的变化，在最后让弯曲变得非常平滑。这让弯曲有了更加强大的效果。

**侧面收缩（Choosing Sides）**

最后一个增强，我们希望能够收缩边界。边界朝着吸收的点消失。直到现在它总是在朝着width值的点消失。添加一个边界属性，我们能够修改这个点在0到width之间。

```
ShaderEffect {
    ...
    property real side: 0.5

    vertexShader: "
        ...
        uniform highp float side;
        ...
        pos.x = mix(qt_Vertex.x, side * width, t * bend);
    "
}
```

![](http://qmlbook.org/_images/geniehalfside.png)

**包装（Packing）**

最后将我们的效果包装起来。将我们吸收效果的代码提取到一个叫做GenieEffect的自定义组件中。它使用ShaderEffect作为根元素。移除掉MouseArea，这不应该放在组件中。绑定minimized属性来触发效果。

```
import QtQuick 2.0

ShaderEffect {
    id: genieEffect
    width: 160; height: width
    anchors.centerIn: parent
    property variant source
    mesh: GridMesh { resolution: Qt.size(10, 10) }
    property real minimize: 0.0
    property real bend: 0.0
    property bool minimized: false
    property real side: 1.0


    ParallelAnimation {
        id: animMinimize
        running: genieEffect.minimized
        SequentialAnimation {
            PauseAnimation { duration: 300 }
            NumberAnimation {
                target: genieEffect; property: 'minimize';
                to: 1; duration: 700;
                easing.type: Easing.InOutSine
            }
            PauseAnimation { duration: 1000 }
        }
        SequentialAnimation {
            NumberAnimation {
                target: genieEffect; property: 'bend'
                to: 1; duration: 700;
                easing.type: Easing.InOutSine }
            PauseAnimation { duration: 1300 }
        }
    }

    ParallelAnimation {
        id: animNormalize
        running: !genieEffect.minimized
        SequentialAnimation {
            NumberAnimation {
                target: genieEffect; property: 'minimize';
                to: 0; duration: 700;
                easing.type: Easing.InOutSine
            }
            PauseAnimation { duration: 1300 }
        }
        SequentialAnimation {
            PauseAnimation { duration: 300 }
            NumberAnimation {
                target: genieEffect; property: 'bend'
                to: 0; duration: 700;
                easing.type: Easing.InOutSine }
            PauseAnimation { duration: 1000 }
        }
    }

    vertexShader: "
        uniform highp mat4 qt_Matrix;
        attribute highp vec4 qt_Vertex;
        attribute highp vec2 qt_MultiTexCoord0;
        uniform highp float height;
        uniform highp float width;
        uniform highp float minimize;
        uniform highp float bend;
        uniform highp float side;
        varying highp vec2 qt_TexCoord0;
        void main() {
            qt_TexCoord0 = qt_MultiTexCoord0;
            highp vec4 pos = qt_Vertex;
            pos.y = mix(qt_Vertex.y, height, minimize);
            highp float t = pos.y / height;
            t = (3.0 - 2.0 * t) * t * t;
            pos.x = mix(qt_Vertex.x, side * width, t * bend);
            gl_Position = qt_Matrix * pos;
        }"
}
```

你现在可以像这样简单的使用这个效果：

```
import QtQuick 2.0

Rectangle {
    width: 480; height: 240
    color: '#1e1e1e'

    GenieEffect {
        source: Image { source: 'assets/lighthouse.jpg' }
        MouseArea {
            anchors.fill: parent
            onClicked: parent.minimized = !parent.minimized
        }
    }
}
```

我们简化了代码，移除了背景矩形框，直接使用图片完成效果，替换了在一个单独的图像元素中加载它。
