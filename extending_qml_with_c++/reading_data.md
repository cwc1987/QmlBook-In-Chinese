# 读取数据（Reading Data）

我们让打开动作打开一个文件对话框。当用户已选择一个文件后，在文件对话框上的```onAccepted```方法被调用。这里我们调用```readDocument()```函数。```readDocument```函数将来自文件对话框的地址设置到我们的```FileIO```对象，并调用```read()```方法。从```FileIO```中加载的文本使用```JSON.parse()```方法解析，并将结果对象作为数据模型直接设置到表格视图上。这样非常方便。

```
Action {
    id: open
    ...
    onTriggered: {
        openDialog.open()
    }
}

...

FileDialog {
    id: openDialog
    onAccepted: {
        root.readDocument()
    }
}

function readDocument() {
    io.source = openDialog.fileUrl
    io.read()
    view.model = JSON.parse(io.text)
}


FileIO {
    id: io
}
```

