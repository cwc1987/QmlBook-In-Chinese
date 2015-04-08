# （间接绑定）Binding Indirectly

与无法直接连接动态创建元素的信号类似，也无法脱离桥接元素（bridge element）与动态创建元素绑定属性。为了绑定任意元素的属性，包括动态创建元素，需要使用绑定元素（Binding element）。

绑定元素（Bindging element）允许你指定一个目标元素（target element），一个属性用来绑定，一个值用来绑定这个属性。通过使用绑定元素（Binding elelemt），例如，绑定一个动态加载元素（dynamically loaded element）的属性。在这个章节中有个入门实例如下所示。

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

通常不会设置一个绑定的目标元素，或者不会有一个给定的属性。当绑定激活时使用绑定元素的属性来限制时间。例如，它可以用来限制用户界面的特定模式。





