# 跟踪动态对象（Tracking Dynamic Objects）

处理动态对象时，通常需要跟踪已创建的对象。另一个常见的功能是能够存储和恢复动态对象的状态。在我们动态填充时，使用链表模型（ListModel）可以非常方便的处理这些问题。

在下面的例子中包含了两种元素，火箭和飞机，能够被用户创建和移动。为了控制整个场景动态创建元素，我们使用一个模型来跟踪项。

**待完成**

**插图**

模型是一个链表模型（ListModel），用已创建的项进行填充。实例化时跟踪对象引用的资源URL。后者不是需要严格跟踪的对象，但是以后会派上用场。

```
import QtQuick 2.0
import "create-object.js" as CreateObject

Item {
    id: root

    ListModel {
        id: objectsModel
    }

    function addPlanet() {
        CreateObject.create("planet.qml", root, itemAdded);
    }

    function addRocket() {
        CreateObject.create("rocket.qml", root, itemAdded);
    }

    function itemAdded(obj, source) {
        objectsModel.append({"obj": obj, "source": source})
    }
```

你可以从上面的例子中看到，create-object.js是一种使得JavaScript引进更加简单的、普遍的方法。创建方法使用了三个参数：一个资源URL，一个根元素和一个完成的回调函数。回调需要两个参数：一个新创建的对象引用和一个资源URL。

这意味着每一次调用addPlanet或者addRocket时，当新建对象被创建完成后悔调用itemAdded函数。后者会将对象的引用和资源URL添加到objectsModel模型中。

可以在很多方面使用objectsModel。在示例中，clearItems函数依赖它。这个函数证明了两个事情。首先，如何遍历模型和执行一个任务，即调用析构函数来移除每一个项。其次，它强调了模型不会更新已经销毁的对象。此外移除模型项已连接的对象问题，模型项的对象属性设置为null，为了补救这个问题，代码显式的清除了已移除对象的模型项。

```
    function clearItems() {
        while(objectsModel.count > 0) {
            objectsModel.get(0).obj.destroy();
            objectsModel.remove(0);
        }
    }
```

通过设置XmlListModel模型的xml属性可以处理XML文档字符串。代码如下，模型展示了反序列化函数。反序列化函数通过设置dsIndex引用模型的第一个项来启动反序列化，然后调用项的创建。然后回调dsItemAdded设置新创建对象的x,y属性，然后更新索引创建下一个对象。

```
    XmlListModel {
        id: xmlModel
        query: "/scene/item"
        XmlRole { name: "source"; query: "source/string()" }
        XmlRole { name: "x"; query: "x/string()" }
        XmlRole { name: "y"; query: "y/string()" }
    }

    function deserialize() {
        dsIndex = 0;
        CreateObject.create(xmlModel.get(dsIndex).source, root, dsItemAdded);
    }

    function dsItemAdded(obj, source) {
        itemAdded(obj, source);
        obj.x = xmlModel.get(dsIndex).x;
        obj.y = xmlModel.get(dsIndex).y;

        dsIndex ++;

        if (dsIndex < xmlModel.count)
            CreateObject.create(xmlModel.get(dsIndex).source, root, dsItemAdded);
    }

    property int dsIndex
```

这个例子演示了如何使用模型跟踪已创建的模型项，和基于信息对模型项序列化和反序列化。这可以用于存储一个动态填充场景，例如窗口部件。在这个例子中，模型被用于跟踪每一个模型项。

另一种解决方案是用于一个场景根项下的子项属性来跟踪子项。然后，这要求项自己知道资源URL用于创建它们自己。这也要求场景只能动态创建子项，以避免序列化或者反序列化静态分配的对象。






