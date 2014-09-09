# 概念（Concept）

粒子模拟的核心是粒子系统（ParticleSystem），它控制了共享时间线。一个场景下可以有多个粒子系统，每个都有自己独立的时间线。一个粒子使用发射器元素（Emitter）发射，使用粒子画笔（ParticlePainter）实现可视化，它可以是一张图片，一个QML项或者一个着色项（shader item）。一个发射器元素（Emitter）也提供向量来控制粒子方向。一个粒子被发送后就再也无法控制。粒子模型提供粒子控制器（Affector），它可以控制已发射粒子的参数。

在一个系统中，粒子可以使用粒子群元素（ParticleGroup）来共享移动时间。默认下，每个例子都属于空（""）组。

![](http://qmlbook.org/_images/particlesystem.png)

* 粒子系统（ParticleSystem）- 管理发射器之间的共享时间线。

* 发射器（Emitter）- 向系统中发射逻辑粒子。

* 粒子画笔（ParticlePainter）- 实现粒子可视化。

* 方向（Direction）- 已发射粒子的向量空间。

* 粒子组（ParticleGroup）- 每个粒子是一个粒子组的成员。

* 粒子控制器（Affector）- 控制已发射粒子。
