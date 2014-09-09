# 着色器元素（Shader Elements）

为了对着色器编程，Qt Quick提供了两个元素。ShaderEffectSource与ShaderEffect。ShaderEffect将会使用自定义的着色器，ShaderEffectSource可以将一个QML元素渲染为一个纹理然后再渲染这个纹理。由于ShaderEffect能够应用自定义的着色器到它的矩形几何形状，并且能够使用在着色器中操作资源。一个资源可以是一个图片，它被作为一个纹理或者着色器资源。

默认下着色器使用这个资源并且不作任何改变进行渲染。

```
import QtQuick 2.0

Rectangle {
    width: 480; height: 240
    color: '#1e1e1e'

    Row {
        anchors.centerIn: parent
        spacing: 20
        Image {
            id: sourceImage
            width: 80; height: width
            source: 'assets/tulips.jpg'
        }
        ShaderEffect {
            id: effect
            width: 80; height: width
            property variant source: sourceImage
        }
        ShaderEffect {
            id: effect2
            width: 80; height: width
            // the source where the effect shall be applied to
            property variant source: sourceImage
            // default vertex shader code
            vertexShader: "
                uniform highp mat4 qt_Matrix;
                attribute highp vec4 qt_Vertex;
                attribute highp vec2 qt_MultiTexCoord0;
                varying highp vec2 qt_TexCoord0;
                void main() {
                    qt_TexCoord0 = qt_MultiTexCoord0;
                    gl_Position = qt_Matrix * qt_Vertex;
                }"
            // default fragment shader code
            fragmentShader: "
                varying highp vec2 qt_TexCoord0;
                uniform sampler2D source;
                uniform lowp float qt_Opacity;
                void main() {
                    gl_FragColor = texture2D(source, qt_TexCoord0) * qt_Opacity;
                }"
        }
    }
}
```

![](http://qmlbook.org/_images/defaultshader.png)

在上边这个例子中，我们在一行中显示了3张图片，第一张是原始图片，第二张使用默认的着色器渲染出来的图片，第三张使用了Qt5源码中默认的顶点与片段着色器的代码进行渲染的图片。

**注意**

**如果你不想看到原始图片，而只想看到被着色器渲染后的图片，你可以设置Image为不可见（visible:false）。着色器仍然会使用图片数据，但是图像元素（Image Element）将不会被渲染。**

让我们仔细看看着色器代码。

```
vertexShader: "
    uniform highp mat4 qt_Matrix;
    attribute highp vec4 qt_Vertex;
    attribute highp vec2 qt_MultiTexCoord0;
    varying highp vec2 qt_TexCoord0;
    void main() {
        qt_TexCoord0 = qt_MultiTexCoord0;
        gl_Position = qt_Matrix * qt_Vertex;
    }"
```

着色器代码来自Qt这边的一个字符串，绑定了顶点着色器（vertexShader）与片段着色器（fragmentShader）属性。每个着色器代码必须有一个main(){....}函数，它将被GPU执行。Qt已经默认提供了以qt_开头的变量。

下面是这些变量简短的介绍：

* uniform-在处理过程中不能够改变的值。

* attribute-连接外部数据

* varying-着色器之间的共享数据

* highp-高精度值

* lowp-低精度值

* mat4-4x4浮点数（float）矩阵

* vec2-包含两个浮点数的向量

* sampler2D-2D纹理

* float-浮点数

可以查看[OpenGL ES 2.0 API Quick Reference Card](http://www.khronos.org/opengles/sdk/docs/reference_cards/OpenGL-ES-2_0-Reference-card.pdf)获得更多信息。

现在我们可以更好的理解下面这些变量：

* qt_Matrix：model-view-projection（模型-视图-投影）矩阵

* qt_Vertex：当前顶点坐标

* qt_MultiTexCoord0：纹理坐标

* qt_TexCoord0：共享纹理坐标

我们已经有可以使用的投影矩阵（projection matrix），当前顶点与纹理坐标。纹理坐标与作为资源（source）的纹理相关。在main()函数中，我们保存纹理坐标，留在后面的片段着色器中使用。每个顶点着色器都需要赋值给gl_Postion，在这里使用项目矩阵乘以顶点，得到我们3D坐标系中的点。

片段着色器从顶点着色器中接收我们的纹理坐标，这个纹理仍然来自我们的QML资源属性（source property）。在着色器代码与QML之间传递变量是如此的简单。此外我们的透明值，在着色器中也可以使用，变量是qt_Opacity。每
个片段着色器需要给gl_FragColor变量赋值，在这里默认着色器代码使用资源纹理（source texture）的像素颜色与透明值相乘。

```
fragmentShader: "
    varying highp vec2 qt_TexCoord0;
    uniform sampler2D source;
    uniform lowp float qt_Opacity;
    void main() {
        gl_FragColor = texture2D(source, qt_TexCoord0) * qt_Opacity;
    }"
```

在后面的例子中，我们将会展示一些简单的着色器例子。首先我们会集中在片段着色器上，然后在回到顶点着色器上。
