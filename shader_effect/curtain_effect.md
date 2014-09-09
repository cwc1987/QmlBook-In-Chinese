# 剧幕效果（Curtain Effect）

在最后的自定义效果例子中，我们将带来一个剧幕效果。这个效果是2011年5月Qt实验室发布的着色器效果中的一部分。目前网址已经转到blog.qt.digia.com，不知道还能不能找到。

![](http://qmlbook.org/_images/curtain.png)

当时我非常喜欢这些效果，剧幕效果是我最喜爱的一个。我喜欢剧幕打开然后遮挡后面的背景对象。

我将代码移植适配到Qt5上，这非常简单。同时我做了一些简化让它能够更好的展示。如果你对整个例子有兴趣，可以访问Qt实验室的博客。

只有一个小组件作为背景，剧幕实际上是一张图片，叫做fabric.jpg，它是ShaderEffect的资源。整个效果使用顶点着色器来摆动剧幕，使用片段着色器提供阴影的效果。下面是一个简单的图片，让你更加容易理解代码。

![](http://qmlbook.org/_images/curtain_diagram.png)

剧幕的波形阴影通过一个在剧幕宽度上的sin曲线使用7的振幅来计算（7*PI=221.99..）另一个重要的部分是摆动，当剧幕打开或者关闭时，使用动画来播放剧幕的topWidth。bottomWidth使用SpringAnimation来跟随topWidth变化。这样我们就能创建出底部摆动的剧幕效果。计算得到的swing提供了摇摆的强度，用来对顶点的y值进行插值。

剧幕效果放在CurtainEffect.qml组件中，fabric图像作为纹理资源。在阴影的使用上没有新的东西加入，唯一不同的是在顶点着色器中操作gl_Postion和片段着色器中操作gl_FragColor。

```
import QtQuick 2.0

ShaderEffect {
    anchors.fill: parent

    mesh: GridMesh {
        resolution: Qt.size(50, 50)
    }

    property real topWidth: open?width:20
    property real bottomWidth: topWidth
    property real amplitude: 0.1
    property bool open: false
    property variant source: effectSource

    Behavior on bottomWidth {
        SpringAnimation {
            easing.type: Easing.OutElastic;
            velocity: 250; mass: 1.5;
            spring: 0.5; damping: 0.05
        }
    }

    Behavior on topWidth {
        NumberAnimation { duration: 1000 }
    }


    ShaderEffectSource {
        id: effectSource
        sourceItem: effectImage;
        hideSource: true
    }

    Image {
        id: effectImage
        anchors.fill: parent
        source: "assets/fabric.jpg"
        fillMode: Image.Tile
    }

    vertexShader: "
        attribute highp vec4 qt_Vertex;
        attribute highp vec2 qt_MultiTexCoord0;
        uniform highp mat4 qt_Matrix;
        varying highp vec2 qt_TexCoord0;
        varying lowp float shade;

        uniform highp float topWidth;
        uniform highp float bottomWidth;
        uniform highp float width;
        uniform highp float height;
        uniform highp float amplitude;

        void main() {
            qt_TexCoord0 = qt_MultiTexCoord0;

            highp vec4 shift = vec4(0.0, 0.0, 0.0, 0.0);
            highp float swing = (topWidth - bottomWidth) * (qt_Vertex.y / height);
            shift.x = qt_Vertex.x * (width - topWidth + swing) / width;

            shade = sin(21.9911486 * qt_Vertex.x / width);
            shift.y = amplitude * (width - topWidth + swing) * shade;

            gl_Position = qt_Matrix * (qt_Vertex - shift);

            shade = 0.2 * (2.0 - shade ) * ((width - topWidth + swing) / width);
        }"

    fragmentShader: "
        uniform sampler2D source;
        varying highp vec2 qt_TexCoord0;
        varying lowp float shade;
        void main() {
            highp vec4 color = texture2D(source, qt_TexCoord0);
            color.rgb *= 1.0 - shade;
            gl_FragColor = color;
        }"
}
```

这个效果在curtaindemo.qml文件中使用。

```
import QtQuick 2.0

Rectangle {
    id: root
    width: 480; height: 240
    color: '#1e1e1e'

    Image {
        anchors.centerIn: parent
        source: 'assets/wiesn.jpg'
    }

    CurtainEffect {
        id: curtain
        anchors.fill: parent
    }

    MouseArea {
        anchors.fill: parent
        onClicked: curtain.open = !curtain.open
    }
}
```

剧幕效果通过自定义的open属性打开。我们使用了一个MouseArea来触发打开和关闭剧幕。
