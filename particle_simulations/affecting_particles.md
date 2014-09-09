# 粒子控制（Affecting Particles）

粒子由粒子发射器发出。在粒子发射出后，发射器无法再改变粒子。粒子控制器允许你控制发射后的粒子参数。

控制器的每个类型使用不同的方法来影响粒子：

* 生命周期（Age）- 修改粒子的生命周期

* 吸引（Attractor）- 吸引粒子朝向指定点

* 摩擦（Friction）- 按当前粒子速度成正比减慢运动

* 重力（Gravity）- 设置一个角度的加速度

* 紊流（Turbulence）- 强制基于噪声图像方式的流动

* 漂移（Wander）- 随机变化的轨迹

* 组目标（GroupGoal）- 改变一组粒子群的状态

* 子粒子（SpriteGoal）- 改变一个子粒子的状态

**生命周期（Age）**

允许粒子老得更快，lifeLeft属性指定了粒子的有多少的生命周期。

```
    Age {
        anchors.horizontalCenter: parent.horizontalCenter
        width: 240; height: 120
        system: particleSystem
        advancePosition: true
        lifeLeft: 1200
        once: true
        Tracer {}
    }
```

在这个例子中，当粒子的生命周期达到1200毫秒后，我们将会缩短上方的粒子的生命周期一次。由于我们设置了advancePosition为true，当粒子的生命周期到达1200毫秒后，我们将会再一次在这个位置看到粒子出现。

![](http://qmlbook.org/_images/age.png)

**吸引（Attractor）**

吸引会将粒子朝指定的点上吸引。这个点使用pointX与pointY来指定，它是与吸引区域的几何形状相对的。strength指定了吸引的力度。在我们的例子中，我们让粒子从左向右运动，吸引放在顶部，有一半运动的粒子会穿过吸引区域。控制器只会影响在它们几何形状内的粒子。这种分离让我们可以同步看到正常的流动与受影响的流动。

```
    Attractor {
        anchors.horizontalCenter: parent.horizontalCenter
        width: 160; height: 120
        system: particleSystem
        pointX: 0
        pointY: 0
        strength: 1.0
        Tracer {}
    }
```

很容易看出上半部分粒子受到吸引。吸引点被设置为吸引区域的左上角（0/0点），吸引力为1.0。

![](http://qmlbook.org/_images/attractor.png)

**摩擦（Friction）**

摩擦控制器使用一个参数（factor）减慢粒子运动，直到达到一个阈值。

```
    Friction {
        anchors.horizontalCenter: parent.horizontalCenter
        width: 240; height: 120
        system: particleSystem
        factor : 0.8
        threshold: 25
        Tracer {}
    }
```

在上部的摩擦区域，粒子被按照0.8的参数（factor）减慢，直到粒子的速度达到25像素每秒。这个阈值像一个过滤器。粒子运动速度高于阈值将会按照给定的参数来减慢它。

![](http://qmlbook.org/_images/friction.png)

**重力（Gravity）**

重力控制器应用在加速度上，在我们的例子中，我们使用一个角度方向将粒子从底部发射到顶部。右边是为控制区域，左边使用重力控制器控制，重力方向为90度方向（垂直向下），梯度值为50。

```
    Gravity {
        width: 240; height: 240
        system: particleSystem
        magnitude: 50
        angle: 90
        Tracer {}
    }
```

左边的粒子试图爬上去，但是稳定向下的加速度将它们按照重力的方向拖拽下来。

![](http://qmlbook.org/_images/gravity.png)

**紊流（Turbulence）**

紊流控制器，对粒子应用了一个混乱映射方向力的矢量。这个混乱映射是由一个噪声图像定义的。可以使用noiseSource属性来定义噪声图像。strength定义了矢量对于粒子运动的影响有多大。

```
    Turbulence {
        anchors.horizontalCenter: parent.horizontalCenter
        width: 240; height: 120
        system: particleSystem
        strength: 100
        Tracer {}
    }
```

在这个例子中，上部区域被紊流影响。它们的运动看起来是不稳定的。不稳定的粒子偏差值来自原路径定义的strength。

![](http://qmlbook.org/_images/turbulence.png)

**漂移（Wander）**

漂移控制器控制了轨迹。affectedParameter属性可以指定哪个参数控制了漂移（速度，位置或者加速度）。pace属性制定了每秒最多改变的属性。yVariance指定了y组件对粒子轨迹的影响。

```
    Wander {
        anchors.horizontalCenter: parent.horizontalCenter
        width: 240; height: 120
        system: particleSystem
        affectedParameter: Wander.Position
        pace: 200
        yVariance: 240
        Tracer {}
    }
```

在顶部漂移控制器的粒子被随机的轨迹改变。在这种情境下，每秒改变粒子y方向的位置200次。

![](http://qmlbook.org/_images/wander.png)
