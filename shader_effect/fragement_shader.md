# 片段着色器（Fragement Shader）

片段着色器调用每个需要渲染的像素。我们将开发一个红色透镜，它将会增加图片的红色通道的值。

**配置场景（Setting up the scene）**

首先我们配置我们的场景，在区域中央使用一个网格显示我们的源图片（source image）。

```
import QtQuick 2.0

Rectangle {
    width: 480; height: 240
    color: '#1e1e1e'

    Grid {
        anchors.centerIn: parent
        spacing: 20
        rows: 2; columns: 4
        Image {
            id: sourceImage
            width: 80; height: width
            source: 'assets/tulips.jpg'
        }
    }
}
```

![](http://qmlbook.org/_images/redlense1.png)

**红色着色器（A red Shader）**

下一步我们添加一个着色器，显示一个红色矩形框。由于我们不需要纹理，我们从顶点着色器中移除纹理。

```
            vertexShader: "
                uniform highp mat4 qt_Matrix;
                attribute highp vec4 qt_Vertex;
                void main() {
                    gl_Position = qt_Matrix * qt_Vertex;
                }"
            fragmentShader: "
                uniform lowp float qt_Opacity;
                void main() {
                    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0) * qt_Opacity;
                }"
```

在片段着色器中，我们简单的给gl_FragColor赋值为vec4(1.0, 0.0, 0.0, 1.0)，它代表红色， 并且不透明（alpha=1.0）。

![](http://qmlbook.org/_images/redlense2.png)

**使用纹理的红色着色器（A red shader with texture）**

现在我们想要将这个红色应用在纹理的每个像素上。我们需要将纹理加回顶点着色器。由于我们不再在顶点着色器中做任何其它的事情，所以默认的顶点着色器已经满足我们的要求。

```
        ShaderEffect {
            id: effect2
            width: 80; height: width
            property variant source: sourceImage
            visible: root.step>1
            fragmentShader: "
                varying highp vec2 qt_TexCoord0;
                uniform sampler2D source;
                uniform lowp float qt_Opacity;
                void main() {
                    gl_FragColor = texture2D(source, qt_TexCoord0) * vec4(1.0, 0.0, 0.0, 1.0) * qt_Opacity;
                }"
        }
```

完整的着色器重新包含我们的源图片作为属性，由于我们没有特殊指定，使用默认的顶点着色器，我没有重写顶点着色器。

在片段着色器中，我们提取纹理片段texture2D(source,qt_TexCoord0)，并且与红色一起应用。

![](http://qmlbook.org/_images/redlense3.png)

**红色通道属性（The red channel property）**

这样的代码用来修改红色通道的值看起来不是很好，所以我们想要将这个值包含在QML这边。我们在ShaderEffect中增加一个redChannel属性，并在我们的片段着色器中申明一个uniform lowpfloat redChannel。这就是从一个着色器代码中标记一个值到QML这边的方法，非常简单。

```
        ShaderEffect {
            id: effect3
            width: 80; height: width
            property variant source: sourceImage
            property real redChannel: 0.3
            visible: root.step>2
            fragmentShader: "
                varying highp vec2 qt_TexCoord0;
                uniform sampler2D source;
                uniform lowp float qt_Opacity;
                uniform lowp float redChannel;
                void main() {
                    gl_FragColor = texture2D(source, qt_TexCoord0) * vec4(redChannel, 1.0, 1.0, 1.0) * qt_Opacity;
                }"
        }
```

为了让这个透镜更真实，我们改变vec4颜色为vec4(redChannel, 1.0, 1.0, 1.0)，这样其它颜色与1.0相乘，只有红色部分使用我们的redChannel变量。

![](http://qmlbook.org/_images/redlense4.png)

**红色通道的动画（The red channel animated**）

由于redChannel属性仅仅是一个正常的属性，我们也可以像其它QML中的属性一样使用动画。我们使用QML属性在GPU上改变这个值，来影响我们的着色器，这真酷！

```
        ShaderEffect {
            id: effect4
            width: 80; height: width
            property variant source: sourceImage
            property real redChannel: 0.3
            visible: root.step>3
            NumberAnimation on redChannel {
                from: 0.0; to: 1.0; loops: Animation.Infinite; duration: 4000
            }

            fragmentShader: "
                varying highp vec2 qt_TexCoord0;
                uniform sampler2D source;
                uniform lowp float qt_Opacity;
                uniform lowp float redChannel;
                void main() {
                    gl_FragColor = texture2D(source, qt_TexCoord0) * vec4(redChannel, 1.0, 1.0, 1.0) * qt_Opacity;
                }"
        }
```

下面是最后的结果。

![](http://qmlbook.org/_images/redlense5.png)

在这4秒内，第二排的着色器红色通道的值从0.0到1.0。图片从没有红色信息（0.0 red）到一个正常的图片（1.0 red）。
