# 动态加载组件（Loading Components Dynamically）

动态加载QML不同组成部分最简单的方法是使用加载元素项（Loader element）。它作为一个占位符项用来加载项。项的加载通过资源属性（source property）或者资源组件（sourceCompontent）属性控制。加载元素项通过给定的URL链接加载项，然后实例化一个组件。

加载元素项（loader）作为一个占位符用于被加载项的加载。它的大小基于被加载项的大小而定，反之亦然。如果加载元素定义了大小，或者通过锚定（anchoring）定义了宽度和高度，被加载项将会被设置为加载元素项的大小。如果加载元素项没有设置大小，它将会根据被加载项的大小而定。

下面例子演示了使用加载元素项（Loader Element）将两个分离的用户界面部分加载到一个相同的空间。这个主意是我们有一个快速拨号界面，可以是数字界面或模拟界面。如下面插图所示。表盘周围的数字不受被加载项影响。

![](http://qmlbook.github.io/_images/loader-analog.png)

![](http://qmlbook.github.io/_images/loader-digital.png)

首先，在应用程序中声明一个加载元素项（Loader element）。注意，资源属性（source property）已经被忽略。这是因为资源取决于当前用户界面状态。

```
Loader {
    id: dialLoader

    anchors.fill: parent
}
```

拨号加载器（dialLoader）的父项的状态属性改变元素驱动根据不同状态加载不同的QML文件。在这个例子中，资源属性（source property）是一个相对文件路径，但是它可以是一个完整的URL链接，通过网络获取加载项。

```
    states: [
        State {
            name: "analog"
            PropertyChanges { target: analogButton; color: "green"; }
            PropertyChanges { target: dialLoader; source: "Analog.qml"; }
        },
        State {
            name: "digital"
            PropertyChanges { target: digitalButton; color: "green"; }
            PropertyChanges { target: dialLoader; source: "Digital.qml"; }
        }
    ]
```

为了使被加载项更加生动，它的速度属性必须根项的速度属性绑定。不能够使用直接绑定来绑定属性，因为项不是总在加载，并且这会随着时间而改变。需要使用一个东西来替换绑定元素（Binding Element）。绑定目标属性（target property），每次加载元素项改变时会触发已加载完成（onLoaded）信号。

```
    Loader {
        id: dialLoader

        anchors.left: parent.left
        anchors.right: parent.right
        anchors.top: parent.top
        anchors.bottom: analogButton.top

        onLoaded: {
            binder.target = dialLoader.item;
        }
    }
    Binding {
        id: binder

        property: "speed"
        value: speed
    }
```

当被加载项加载完成后，加载完成信号（onLoaded）会触发加载QML的动作。类似的，QML完成加载以来与组建构建完成（Component.onCompleted）信号。所有组建都可以使用这个信号，无论它们何时加载。例如，当整个用户界面完成加载后，整个应用程序的根组建可以使用它来启动自己。



