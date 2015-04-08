# （动态加载和实例化项）Dynamically Loading and Instantiating Items

加载一块QML代码时，它首先会被解释执行为一个组件。这一步包含了加载依赖和验证代码。QML的来源可以是本地文件，Qt资源文件，或者一个指定的URL网络地址。这意味着加载时间不确定。例如一个不需要加载任何依赖位于内存（RAM）中的Qt资源文件加载非常快，或者一个需要加载多种依赖位于一个缓慢的服务器中加载需要很长的时间。

创建一个组件的状态可以用来跟踪它的状态属性。可以使用的状态值包括组件为空（Component.NULL）、组件加载中（Component.Loading）、组件可用（Component.Ready）和组件错误（Component.Error）。从空（NULL）状态到加载中（Loading）再到可用（Ready）通常是一个工作流。在任何一个阶段状态都可以变为错误（Error）。在这种情况下，组件无法被用来创建新的对象实例。Component.errorString()函数用来检索用户可读的错误描述。

当加载连接缓慢的组件时，可以使用进度（progress）属性。它的范围从0.0意味着为加载任何东西，到1.0表明加载已完成。当组件的状态改变为可用（Ready）时，组件可以用实例化对象。下面的代码演示了如何实现这样的方法，考虑组件变为可用或者创建失败，同时组件加载时间可能会比较慢。

```
var component;

function createImageObject() {
    component = Qt.createComponent("dynamic-image.qml");
    if (component.status === Component.Ready || component.status === Component.Error)
        finishCreation();
    else
        component.statusChanged.connect(finishCreation);
}

function finishCreation() {
    if (component.status === Component.Ready)
    {
        var image = component.createObject(root, {"x": 100, "y": 100});
        if (image == null)
            console.log("Error creating image");
    }
    else if (component.status === Component.Error)
        console.log("Error loading component:", component.errorString());
}
```

上面的代码是源文件中的JavaScript代码，来自main QML文件。

```
import QtQuick 2.0
import "create-component.js" as ImageCreator

Item {
    id: root

    width: 1024
    height: 600

    Component.onCompleted: ImageCreator.createImageObject();
}
```

一个组件的创建对象（createObject）对象函数用于创建一个实例化对象，如上所示。这不仅仅用于动态动态加载组件，也用语言QML代码中的组件内联。这样产生的对象可以像其它的对象一样用于QML场景中。唯一的不同是这些对象没有id。

创建对象（createObject）函数接受两个参数。第一个参数是父对象。第二个参数是按照格式{"name": value, "name": value}组成的一串属性和值。下面的例子演示了这种用法。注意，属性参数是可选的。

```
var image = component.createObject(root, {"x": 100, "y": 100});
```

**注意**

**一个动态创建的组件实例不同于一个内联组件元素（in-line Component element）。内联组件元素也提供了函数用来动态实例化对象。**





