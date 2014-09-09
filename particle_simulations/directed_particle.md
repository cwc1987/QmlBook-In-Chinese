# 粒子方向（Directed Particle）

我们已经看到了粒子的旋转，但是我们的粒子需要一个轨迹。轨迹由速度或者粒子随机方向的加速度指定，也可以叫做矢量空间。

有多种可用矢量空间用来定义粒子的速度或加速度：

* 角度方向（AngleDirection）- 使用角度的方向变化。

* 点方向（PointDirection）-  使用x,y组件组成的方向变化。

* 目标方向（TargetDirection）- 朝着目标点的方向变化。

![](http://qmlbook.org/_images/particle_directions.png)

让我们在场景下试着用速度方向将粒子从左边移动到右边。

首先使用角度方向（AngleDirection）。我们使用AngleDirection元素作为我们的发射器（Emitter）的速度属性：

```
velocity: AngleDirection { }
```

粒子的发射将会使用指定的角度属性。角度值在0到360度之间，0度代表指向右边。在我们的例子中，例子将会移动到右边，所以0度已经指向右边方向。粒子的角度变化在+/-15度之间：

```
velocity: AngleDirection {
    angle: 0
    angleVariation: 15
}
```

现在我们已经设置了方向，下面是指定粒子的速度。它由一个梯度值定义，这个梯度值定义了每秒像素的变化。正如我们设置大约640像素，梯度值为100，看起来是一个不错的值。这意味着平均一个6.4秒生命周期的粒子可以穿越我们看到的区域。为了让粒子的穿越看起来更加有趣，我们使用magnitudeVariation来设置梯度值的变化，这个值是我们的梯度值的一半：

```
velocity: AngleDirection {
    ...
    magnitude: 100
    magnitudeVariation: 50
}
```

![](http://qmlbook.org/_images/angledirection.png)

下面是完整的源码，平均的生命周期被设置为6..4秒。我们设置发射器的宽度和高度为1个像素，这意味着所有的粒子都从相同的位置发射出去，然后基于我们给定的轨迹运动。

```
    Emitter {
        id: emitter
        anchors.left: parent.left
        anchors.verticalCenter: parent.verticalCenter
        width: 1; height: 1
        system: particleSystem
        lifeSpan: 6400
        lifeSpanVariation: 400
        size: 32
        velocity: AngleDirection {
            angle: 0
            angleVariation: 15
            magnitude: 100
            magnitudeVariation: 50
        }
    }
```

那么加速度做些什么？加速度是每个粒子加速度矢量，它会在运动的时间中改变速度矢量。例如我们做一个星星按照弧形运动的轨迹。我们将会改变我们的速度方向为-45度，然后移除变量，可以得到一个更连贯的弧形轨迹：

```
velocity: AngleDirection {
    angle: -45
    magnitude: 100
}
```

加速度的方向为90度（向下），加速度为速度的四分之一：

```
acceleration: AngleDirection {
    angle: 90
    magnitude: 25
}
```

结果是中间左方到右下的一个弧。

![](http://qmlbook.org/_images/angledirection2.png)
这个值是在不断的尝试与错误中发现的。

下面是发射器完整的代码。

```
    Emitter {
        id: emitter
        anchors.left: parent.left
        anchors.verticalCenter: parent.verticalCenter
        width: 1; height: 1
        system: particleSystem
        emitRate: 10
        lifeSpan: 6400
        lifeSpanVariation: 400
        size: 32
        velocity: AngleDirection {
            angle: -45
            angleVariation: 0
            magnitude: 100
        }
        acceleration: AngleDirection {
            angle: 90
            magnitude: 25
        }
    }
```

在下一个例子中，我们将使用点方向（PointDirection）矢量空间来再一次演示粒子从左到右的运动。

一个点方向（PointDirection）是由x和y组件组成的矢量空间。例如，如果你想粒子以45度的矢量运动，你需要为x，y指定相同的值。

在我们的例子中，我们希望粒子在从左到右的例子中建立一个15度的圆锥。我们指定一个坐标方向（PointDirection）作为我们速度矢量空间：

```
velocity: PointDirection { }
```

为了达到运动速度每秒100个像素，我们设置x为100,。15度角（90度的1/6）,我们指定y变量为100/6：

```
velocity: PointDirection {
    x: 100
    y: 0
    xVariation: 0
    yVariation: 100/6
}
```

结果是粒子的运动从左到右构成了一个15度的圆锥。

![](http://qmlbook.org/_images/pointdirection.png)

现在是最后一个方案，目标方向（TargetDirection）。目标方向允许我们指定发射器或者一个QML项的x,y坐标值。当一个QML项的中心点成为一个目标点时，你可以指定目标变化值是x目标值的1/6来完成一个15度的圆锥：

```
velocity: TargetDirection {
    targetX: 100
    targetY: 0
    targetVariation: 100/6
    magnitude: 100
}
```

**注意**

**当你期望发射粒子朝着指定的x,y坐标值流动时，目标方向是非常好的方案。**

我没有再贴出结果图，因为它与前一个结果相同，取而代之的有一个问题留给你。

在下图的红色和绿色圆指定每个目标项的目标方向速度的加速属性。每个目标方向有相同的参数。那么哪一个负责速度，哪一个负责加速度？

![](http://qmlbook.org/_images/directionquest.png)
