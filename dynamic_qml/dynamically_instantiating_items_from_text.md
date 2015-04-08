# 从文本中动态实例化项（Dynamically Instantiating Items from Text）

有时，可以很方便的从QML文本字符串中实例化一个对象。别的不说，这比将代码从源文件中分离后拿出来快。为了实现这个功能，需要使用Qt.createQmlObject函数。

这个函数接受三个参数：qml，parent和filepath。qml参数包含了用来实例化的QML代码字符串。parent参数为新创建的对象提供了一个父对象。filepath参数用于存储创建对象时的错误报告。这个函数的结果返回一个新的对象或者一个NULL。

**警告**

**createQmlObject函数通常会立即返回结果。为了成功调用这个函数，所有的依赖调用需要保证已经被加载。这意味着如果函数调用了未加载的组件，这个调用就会失败并且返回null。为了更好的处理这个问题，必须使用createComponent/createObject方法。**

使用Qt.createQmlObject函数创建对象与其它的动态创建对象类似。这说明与其它创建的QML对象一样，也没有id。在下面的例子中，当根元素创建完成后，从内联QML代码中实例化了一个新的矩形元素（Rectangle element）。

```
import QtQuick 2.0

Item {
    id: root

    width: 1024
    height: 600

    function createItem() {
        Qt.createQmlObject("import QtQuick 2.0; Rectangle { x: 100; y: 100; width: 100;
    height:100; color: \"blue\" }", root, "dynamicItem");
    }

    Component.onCompleted: root.createItem();
}
```



