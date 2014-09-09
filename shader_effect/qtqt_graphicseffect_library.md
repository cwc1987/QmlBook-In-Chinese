# Qt图像效果库（Qt GraphicsEffect Library）

图像效果库是一个着色器效果的集合，是由Qt开发者提供制作的。它是一个很好的工具，你可以将它应用在你的程序中，它也是一个学习如何创建着色器的例子。

图像效果库附带了一个手动测试平台，这个工具可以帮助你测试发现不同的效果
测试工具在$QTDIR/qtgraphicaleffects/tests/manual/testbed下。

![](http://qmlbook.org/_images/graphicseffectstestbed.png)

效果库包含了大约20种效果，下面是效果列表和一些简短的描述。

| 种类 | 效果 | 描述 |
| -- | -- | -- |
| 混合（Blend）| 混合（Blend） | 使用混合模式合并两个资源项 |
| 颜色（Color） | 亮度与对比度（BrightnessContrast） | 调整亮度与对比度 |
|  | 着色（Colorize）| 设置HSL颜色空间颜色 |
| | 颜色叠加（ColorOverlay） | 应用一个颜色层 |
| | 降低饱和度（Desaturate） | 减少颜色饱和度 |
| | 伽马调整（GammaAdjust） | 调整发光度 |
| | 色调饱和度（HueSaturation） | 调整HSL颜色空间颜色 |
| | 色阶调整（LevelAdjust） | 调整RGB颜色空间颜色 |
| 渐变（Gradient） | 圆锥渐变（ConicalGradient） | 绘制一个圆锥渐变 |
| | 线性渐变（LinearGradient） | 绘制一个线性渐变 |
| | 射线渐变（RadialGradient） | 绘制一个射线渐变 |
| 失真（Distortion） | 置换（Displace） | 按照指定的置换源移动源项的像素 |
| 阴影（Drop Shadow） | 阴影 （DropShadow） | 绘制一个阴影 |
| | 内阴影（InnerShadow） | 绘制一个内阴影 |
| 模糊 （Blur）| 快速模糊（FastBlur） | 应用一个快速模糊效果 |
| | 高斯模糊（GaussianBlur） | 应用一个高质量模糊效果 |
| | 蒙版模糊（MaskedBlur）| 应用一个多种强度的模糊效果 |
| | 递归模糊（RecursiveBlur） | 重复模糊，提供一个更强的模糊效果 |
| 运动模糊（Motion Blur） | 方向模糊（DirectionalBlur） | 应用一个方向的运动模糊效果 |
| | 放射模糊（RadialBlur） | 应用一个放射运动模糊效果 |
| | 变焦模糊（ZoomBlur） | 应用一个变焦运动模糊效果 |
| 发光（Glow）| 发光（Glow） | 绘制一个外发光效果 |
| | 矩形发光（RectangularGlow） | 绘制一个矩形外发光效果 |
| 蒙版（Mask）| 透明蒙版（OpacityMask） | 使用一个源项遮挡另一个源项 |
| | 阈值蒙版（ThresholdMask） | 使用一个阈值，一个源项遮挡另一个源项 |

下面是一个使用快速模糊效果的例子：

```
import QtQuick 2.0
import QtGraphicalEffects 1.0

Rectangle {
    width: 480; height: 240
    color: '#1e1e1e'

    Row {
        anchors.centerIn: parent
        spacing: 16

        Image {
            id: sourceImage
            source: "assets/tulips.jpg"
            width: 200; height: width
            sourceSize: Qt.size(parent.width, parent.height)
            smooth: true
        }

        FastBlur {
            width: 200; height: width
            source: sourceImage
            radius: blurred?32:0
            property bool blurred: false

            Behavior on radius {
                NumberAnimation { duration: 1000 }
            }

            MouseArea {
                id: area
                anchors.fill: parent
                onClicked: parent.blurred = !parent.blurred
            }
        }
    }
}
```

左边是原图片。点击右边的图片将会触发blurred属性，模糊在1秒内从0到32。左边显示模糊后的图片。

![](http://qmlbook.org/_images/fastblur.png)
