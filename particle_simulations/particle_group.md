# 粒子组（Particle Group）

在本章开始时，我们已经介绍过粒子组了，默认下，粒子都属于空组（""）。使用GroupGoal控制器可以改变粒子组。为了实现可视化，我们创建了一个烟花示例，火箭进入，在空中爆炸形成壮观的烟火。

![](http://qmlbook.org/_images/firework_teaser.png)

这个例子分为两部分。第一部分叫做“发射时间（Launch Time）”连接场景，加入粒子组，第二部分叫做“爆炸烟花（Let there be firework）”，专注于粒子组的变化。

让我们看看这两部分。

**发射时间（Launch Time）**

首先我们创建一个典型的黑色场景：

```
import QtQuick 2.0
import QtQuick.Particles 2.0

Rectangle {
    id: root
    width: 480; height: 240
    color: "#1F1F1F"
    property bool tracer: false
}
```

tracer使用被用作场景追踪的开关，然后定义我们的粒子系统：

```
ParticleSystem {
    id: particleSystem
}
```

我们添加两种粒子图片画笔（一个用于火箭，一个用于火箭喷射烟雾）：

```
ImageParticle {
    id: smokePainter
    system: particleSystem
    groups: ['smoke']
    source: "assets/particle.png"
    alpha: 0.3
    entryEffect: ImageParticle.None
}

ImageParticle {
    id: rocketPainter
    system: particleSystem
    groups: ['rocket']
    source: "assets/rocket.png"
    entryEffect: ImageParticle.None
}
```

你可以看到在这些画笔定义中，它们使用groups属性来定义粒子的归属。只需要定义一个名字，Qt Quick将会隐式的创建这个分组。

现在我们需要将这些火箭发射到空中。我们在场景底部创建一个粒子发射器，将速度设置为朝上的方向。为了模拟重力，我们设置一个向下的加速度：

```
Emitter {
    id: rocketEmitter
    anchors.bottom: parent.bottom
    width: parent.width; height: 40
    system: particleSystem
    group: 'rocket'
    emitRate: 2
    maximumEmitted: 4
    lifeSpan: 4800
    lifeSpanVariation: 400
    size: 32
    velocity: AngleDirection { angle: 270; magnitude: 150; magnitudeVariation: 10 }
    acceleration: AngleDirection { angle: 90; magnitude: 50 }
    Tracer { color: 'red'; visible: root.tracer }
}
```

发射器属于'rocket'粒子组，与我们的火箭粒子画笔相同。通过粒子组将它们联系在一起。发射器将粒子发射到'rocket'粒子组中，火箭画笔将会绘制它们。

对于烟雾，我们使用一个追踪发射器，它将会跟在火箭的后面。我们定义'smoke'组，并且它会跟在'rocket'粒子组后面：

```
TrailEmitter {
    id: smokeEmitter
    system: particleSystem
    emitHeight: 1
    emitWidth: 4
    group: 'smoke'
    follow: 'rocket'
    emitRatePerParticle: 96
    velocity: AngleDirection { angle: 90; magnitude: 100; angleVariation: 5 }
    lifeSpan: 200
    size: 16
    sizeVariation: 4
    endSize: 0
}
```

向下模拟从火箭里面喷射出的烟。emitHeight与emitWidth指定了围绕跟随在烟雾粒子发射后的粒子。如果不指定这个值，跟随的粒子将会被拿掉，但是对于这个例子，我们想要提升显示效果，粒子流从一个接近于火箭尾部的中间点发射出。

如果你运行这个例子，你会发现一些火箭正常飞起，一些火箭却飞出场景。这不是我们想要的，我们需要在它们离开场景前让他们慢下来，这里可以使用摩擦控制器来设置一个最小阈值：

```
Friction {
    groups: ['rocket']
    anchors.top: parent.top
    width: parent.width; height: 80
    system: particleSystem
    threshold: 5
    factor: 0.9
}
```

在摩擦控制器中，你也需要定义哪个粒子组受控制器影响。当火箭经过从顶部向下80像素的区域时，所有的火箭将会以0.9的factor减慢（你可以试试100，你会发现它们立即停止了），直到它们的速度达到每秒5个像素。随着火箭粒子向下的加速度继续生效，火箭开始向地面下沉，直到它们的生命周期结束。

由于在空气中向上运动是非常困难的，并且非常不稳定，我们在火箭上升时模拟一些紊流：

```
Turbulence {
    groups: ['rocket']
    anchors.bottom: parent.bottom
    width: parent.width; height: 160
    system: particleSystem
    strength: 25
    Tracer { color: 'green'; visible: root.tracer }
}
```

当然，紊流控制器也需要定义它会影响哪些粒子组。紊流控制器的区域从底部向上160像素（直到摩擦控制器边界上），它们也可以相互覆盖。

当你运行程序时，你可以看到火箭开始上升，然后在摩擦控制器区域开始减速，向下的加速度仍然生效，火箭开始后退。下一步我们开始制作爆炸烟花。

![](http://qmlbook.org/_images/firework_rockets.png)

**注意**

**使用tracers跟踪区域可以显示场景中的不同区域。火箭粒子发射的红色区域，蓝色区域是紊流控制器区域，最后在绿色的摩擦控制器区域减速，并且再次下降是由于向下的加速度仍然生效。**

**爆炸烟花（Let there be fireworks）**

让火箭变成美丽的烟花，我们需要添加一个粒子组来封装这个变化：

```
ParticleGroup {
    name: 'explosion'
    system: particleSystem
}
```

我们使用GroupGoal控制器来改变粒子组。这个组控制器被放置在屏幕中间垂直线附近，它将会影响'rocket'粒子组。使用groupGoal属性，我们设置目标组改变为我们之前定义的'explosion'组：

```
GroupGoal {
    id: rocketChanger
    anchors.top: parent.top
    width: parent.width; height: 80
    system: particleSystem
    groups: ['rocket']
    goalState: 'explosion'
    jump: true
    Tracer { color: 'blue'; visible: root.tracer }
}
```

jump属性定义了粒子组的变化是立即变化而不是在某个时间段后变化。

**注意**

**在Qt5的alpha发布版中，粒子组的持续改变无法工作，有好的建议吗？**

由于火箭粒子变为我们的爆炸粒子，当火箭粒子进入GroupGoal控制器区域时，我们需要在粒子组中添加一个烟花：

```
// inside particle group
TrailEmitter {
    id: explosionEmitter
    anchors.fill: parent
    group: 'sparkle'
    follow: 'rocket'
    lifeSpan: 750
    emitRatePerParticle: 200
    size: 32
    velocity: AngleDirection { angle: -90; angleVariation: 180; magnitude: 50 }
}
```

爆炸释放粒子到'sparkle'粒子组。我们稍后会定义这个组的粒子画笔。轨迹发射器跟随火箭粒子每秒发射200个火箭爆炸粒子。粒子的方向向上，并改变180度。

由于向'sparkle'粒子组发射粒子，我们需要定义一个粒子画笔用于绘制这个组的粒子：

```
ImageParticle {
    id: sparklePainter
    system: particleSystem
    groups: ['sparkle']
    color: 'red'
    colorVariation: 0.6
    source: "assets/star.png"
    alpha: 0.3
}
```

闪烁的烟花是红色的星星，使用接近透明的颜色来渲染出发光的效果。

为了使烟花更加壮观，我们也需要添加给我们的粒子组添加第二个轨迹发射器，它向下发射锥形粒子：

```
// inside particle group
TrailEmitter {
    id: explosion2Emitter
    anchors.fill: parent
    group: 'sparkle'
    follow: 'rocket'
    lifeSpan: 250
    emitRatePerParticle: 100
    size: 32
    velocity: AngleDirection { angle: 90; angleVariation: 15; magnitude: 400 }
}
```

其它的爆炸轨迹发射器与这个设置类似，就这样。

下面是最终结果。

![](http://qmlbook.org/_images/firework_final.png)

下面是火箭烟花的所有代码。

```
import QtQuick 2.0
import QtQuick.Particles 2.0

Rectangle {
    id: root
    width: 480; height: 240
    color: "#1F1F1F"
    property bool tracer: false

    ParticleSystem {
        id: particleSystem
    }

    ImageParticle {
        id: smokePainter
        system: particleSystem
        groups: ['smoke']
        source: "assets/particle.png"
        alpha: 0.3
    }

    ImageParticle {
        id: rocketPainter
        system: particleSystem
        groups: ['rocket']
        source: "assets/rocket.png"
        entryEffect: ImageParticle.Fade
    }

    Emitter {
        id: rocketEmitter
        anchors.bottom: parent.bottom
        width: parent.width; height: 40
        system: particleSystem
        group: 'rocket'
        emitRate: 2
        maximumEmitted: 8
        lifeSpan: 4800
        lifeSpanVariation: 400
        size: 32
        velocity: AngleDirection { angle: 270; magnitude: 150; magnitudeVariation: 10 }
        acceleration: AngleDirection { angle: 90; magnitude: 50 }
        Tracer { color: 'red'; visible: root.tracer }
    }

    TrailEmitter {
        id: smokeEmitter
        system: particleSystem
        group: 'smoke'
        follow: 'rocket'
        size: 16
        sizeVariation: 8
        emitRatePerParticle: 16
        velocity: AngleDirection { angle: 90; magnitude: 100; angleVariation: 15 }
        lifeSpan: 200
        Tracer { color: 'blue'; visible: root.tracer }
    }

    Friction {
        groups: ['rocket']
        anchors.top: parent.top
        width: parent.width; height: 80
        system: particleSystem
        threshold: 5
        factor: 0.9

    }

    Turbulence {
        groups: ['rocket']
        anchors.bottom: parent.bottom
        width: parent.width; height: 160
        system: particleSystem
        strength:25
        Tracer { color: 'green'; visible: root.tracer }
    }


    ImageParticle {
        id: sparklePainter
        system: particleSystem
        groups: ['sparkle']
        color: 'red'
        colorVariation: 0.6
        source: "assets/star.png"
        alpha: 0.3
    }

    GroupGoal {
        id: rocketChanger
        anchors.top: parent.top
        width: parent.width; height: 80
        system: particleSystem
        groups: ['rocket']
        goalState: 'explosion'
        jump: true
        Tracer { color: 'blue'; visible: root.tracer }
    }

    ParticleGroup {
        name: 'explosion'
        system: particleSystem

        TrailEmitter {
            id: explosionEmitter
            anchors.fill: parent
            group: 'sparkle'
            follow: 'rocket'
            lifeSpan: 750
            emitRatePerParticle: 200
            size: 32
            velocity: AngleDirection { angle: -90; angleVariation: 180; magnitude: 50 }
        }

        TrailEmitter {
            id: explosion2Emitter
            anchors.fill: parent
            group: 'sparkle'
            follow: 'rocket'
            lifeSpan: 250
            emitRatePerParticle: 100
            size: 32
            velocity: AngleDirection { angle: 90; angleVariation: 15; magnitude: 400 }
        }
    }
}
```
