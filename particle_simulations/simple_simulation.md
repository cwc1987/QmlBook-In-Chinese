# 简单的模拟（Simple Simulation）

让我们从一个简单的模拟开始学习。Qt Quick使用简单的粒子渲染非常简单。下面是我们需要的：

* 绑定所有元素到一个模拟的粒子系统（ParticleSystem）。

* 一个向系统发射粒子的发射器（Emitter）。

* 一个ParticlePainter派生元素，用来实现粒子的可视化。

```
import QtQuick 2.0
import QtQuick.Particles 2.0

Rectangle {
    id: root
    width: 480; height: 160
    color: "#1f1f1f"

    ParticleSystem {
        id: particleSystem
    }

    Emitter {
        id: emitter
        anchors.centerIn: parent
        width: 160; height: 80
        system: particleSystem
        emitRate: 10
        lifeSpan: 1000
        lifeSpanVariation: 500
        size: 16
        endSize: 32
        Tracer { color: 'green' }
    }

    ImageParticle {
        source: "assets/particle.png"
        system: particleSystem
    }
}
```

例子的运行结果如下所示：

![](http://qmlbook.org/_images/simpleparticles.png)

我们使用一个80x80的黑色矩形框作为我们的根元素和背景。然后我们定义一个粒子系统（ParticleSystem）。这通常是粒子系统绑定所有元素的第一步。下一个元素是发射器（Emitter），它定义了基于矩形框的发射区域和发射粒子的基础属性。发射器使用system属性与粒子系统进行绑定。

在这个例子中，发射器每秒发射10个粒子（emitRate:10）到发射器的区域，每个粒子的生命周期是1000毫秒（lifeSpan:1000），一个已发射粒子的生命周期变化是500毫秒（lifeSpanVariation:500）。一个粒子开始的大小是16个像素（size:16），生命周期结束时的大小是32个像素（endSize:32）。

绿色边框的矩形是一个跟踪元素，用来显示发射器的几何形状。这个可视化展示了粒子在发射器矩形框内发射，但是渲染效果不被限制在发射器的矩形框内。渲染位置依赖于粒子的寿命和方向。这将帮助我们更加清楚的知道如何改变粒子的方向。

发射器发射逻辑粒子。一个逻辑粒子的可视化使用粒子画笔（ParticlePainter）来实现，在这个例子中我们使用了图像粒子（ImageParticle），使用一个图片链接作为源属性。图像粒子也有其它的属性用来控制粒子的外观。

* 发射频率（emitRate）- 每秒粒子发射数（默认为10个）。

* 生命周期（lifeSpan）- 粒子持续时间（单位毫秒，默认为1000毫秒）。

* 初始大小（size），结束大小（endSize）- 粒子在它的生命周期的开始和结束时的大小（默认为16像素）。

改变这些属性将会彻底改变显示结果：

```
    Emitter {
        id: emitter
        anchors.centerIn: parent
        width: 20; height: 20
        system: particleSystem
        emitRate: 40
        lifeSpan: 2000
        lifeSpanVariation: 500
        size: 64
        sizeVariation: 32
        Tracer { color: 'green' }
    }
```

增加发射频率为40，生命周期增加到2秒，开始大小为64像素，结束大小减少到32像素。

![](http://qmlbook.org/_images/simpleparticles2.png)

增加结束大小（endSize）可能会导致白色的背景出现。请注意粒子只有发射被限制在发射器定义的区域内，而粒子渲染是不会考虑这个参数的。
