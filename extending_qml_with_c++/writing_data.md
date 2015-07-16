# 写入数据（Writing Data）

我们连接保存动作到saveDocument()函数来保存文档。保存文档函数从视图中取出模型，模型是一个JS对象，并使用JSON.stringify()函数将它转换为一个字符串。将结果字符串设置到FileIO对象的文本属性中，并调用write()来保存数据到磁盘中。在stringify函数上参数null和4将会使用4个空格缩进格式化JSON数据结果。这只是为了保存文档更好阅读。

```
Action {
    id: save
    ...
    onTriggered: {
        saveDocument()
    }
}

function saveDocument() {
    var data = view.model
    io.text = JSON.stringify(data, null, 4)
    io.write()
}

FileIO {
    id: io
}
```
