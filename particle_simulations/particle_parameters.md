# 粒子参数（Particle Parameters）

我们已经知道通过改变发射器的行为就可以改变我们的粒子模拟。粒子画笔被用来绘制每一个粒子。
回到我们之前的粒子中，我们更新一下我们的图片粒子画笔（ImageParticle）。首先我们改变粒子图片为一个小的星形图片：

```
ImageParticle {
    ...
    source: 'assets/star.png'
}
```

粒子使用金色来进行初始化，不同的粒子颜色变化范围为+/- 20%。

```
color: '#FFD700'
colorVariation: 0.2
```

为了让场景更加生动，我们需要旋转粒子。每个粒子首先按顺时针旋转15度，不同的粒子在+/-5度之间变化。每个例子会不断的以每秒45度旋转。每个粒子的旋转速度在+/-15度之间变化：

```
rotation: 15
rotationVariation: 5
rotationVelocity: 45
rotationVelocityVariation: 15
```

最后，我们改变粒子的入场效果。 这个效果是粒子产生时的效果，在这个例子中，我们希望使用一个缩放效果：

```
entryEffect: ImageParticle.Scale
```

现在我们可以看到旋转的星星出现在我们的屏幕上。

![](http://qmlbook.org/_images/particleparameters.png)

下面是我们如何改变图片粒子画笔的代码段。

```
    ImageParticle {
        source: "assets/star.png"
        system: particleSystem
        color: '#FFD700'
        colorVariation: 0.2
        rotation: 0
        rotationVariation: 45
        rotationVelocity: 15
        rotationVelocityVariation: 15
        entryEffect: ImageParticle.Scale
    }
```
