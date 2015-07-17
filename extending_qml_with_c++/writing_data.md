# 写入数据（Writing Data）

我们连接保存动作到```saveDocument()```函数来保存文档。保存文档函数从视图中取出模型，模型是一个JS对象，并使用```JSON.stringify()```函数将它转换为一个字符串。将结果字符串设置到```FileIO```对象的文本属性中，并调用```write()```来保存数据到磁盘中。在```stringify```函数上参数```null```和```4```将会使用4个空格缩进格式化JSON数据结果。这只是为了保存文档更好阅读。

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

从根本上说，这个应用程序就是读取，写入和现实一个JSON文档。考虑下如果使用XML格式读取和写入，会花多少时间。使用JSON格式你只需要读取/写入一个文本文件或者发送/接收一个文本缓存。

![](http://qmlbook.github.io/_images/cityui_table.png)
