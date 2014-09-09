# 粒子画笔（Particle Painter）

到目前为止我们只使用了基于粒子画笔的图像来实现粒子可视化。Qt也提供了一些其它的粒子画笔：

* 粒子项（ItemParticle）：基于粒子画笔的代理

* 自定义粒子（CustomParticle）：基于粒子画笔的着色器

粒子项可以将QML元素项作为粒子发射。你需要制定自己的粒子代理。

```
    ItemParticle {
        id: particle
        system: particleSystem
        delegate: itemDelegate
    }
```

在这个例子中，我们的代理是一个随机图片（使用Math.random()完成），有着白色边框和随机大小。

```
    Component {
        id: itemDelegate
        Rectangle {
            id: container
            width: 32*Math.ceil(Math.random()*3); height: width
            color: 'white'
            Image {
                anchors.fill: parent
                anchors.margins: 4
                source: 'assets/fruits'+Math.ceil(Math.random()*10)+'.jpg'
            }
        }
    }
```

每秒发出四个粒子，每个粒子拥有4秒的生命周期。粒子自动淡入淡出。

![](http://qmlbook.org/_images/itemparticle.png)

对于更多的动态情况，也可以由你自己创建一个子项，让粒子系统来控制它，使用take(item, priority)来完成。粒子系统控制你的粒子就像控制普通的粒子一样。你可以使用give(item)来拿回子项的控制权。你也可以操作子项粒子，甚至可以使用freeze(item)来停止它，使用unfreeze(item)来恢复它。
