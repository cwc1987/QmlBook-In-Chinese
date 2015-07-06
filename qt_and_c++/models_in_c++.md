# C++数据模型（Models in C++）

在QML中的数据模型为链表视图，路径视图和其它需要为模型中的每个子项创建一个代理引用的视图提供数据。视图只创建可是区域内或者缓冲范围内的引用。这使得即使包含成千上万的子项模型仍然可以保持流畅的用户界面。代理扮演了用来渲染模型子项数据的模板。总之：视图使用代理作为模板来渲染模型中的子项。模型为视图提供数据。

当你不想使用C++时，你可以在QML环境中定义模型，你有多重方法为一个视图提供模型。使用C++操作数据或者使用包含了大型数据的C++模型比在QML环境中达到相同目的更加稳定可靠。但是当你值需要少量数据时，QML模型时非常适合的。

```
ListView {
    // using a integer as model
    model: 5
    delegate: Text { text: 'index: ' + index }
}

ListView {
    // using a JS array as model
    model: ['A', 'B', 'C', 'D', 'E']
    delegate: Text { 'Char['+ index +']: ' + modelData }
}

ListView {
    // using a dynamic QML ListModel as model
    model: ListModel {
        ListElement { char: 'A' }
        ListElement { char: 'B' }
        ListElement { char: 'C' }
        ListElement { char: 'D' }
        ListElement { char: 'E' }
    }
    delegate: Text { 'Char['+ index +']: ' + model.char }
}
```

QML视图知道如何操作不同的模型。对于来自C++的模型需要遵循一个特定的协议。这个协议与动态行为一起被定义在一个API（```QAbstractItemModel```）中。这个API时为桌面窗口开发的，它可以灵活的作为一个树的基础或者多列表格或者链表。在QML中我们通常值使用API的链表版本（```QAbstractListModel```）。API包含了一些需要强制实现的函数，另外一些函数时可选的。可选部分通常是用来动态添加或者删除数据。



