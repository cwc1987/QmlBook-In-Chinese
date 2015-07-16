# 使用动作（Using Actions）

为了更好的使用/复用我们的命令，我们使用QML```Action```类型。这将允许我们在后面可以使用相同的动作，也可以用于潜在的工具栏。打开，保存和退出动作是标准动作。打开和保存动作不会包含任何逻辑，我们后面再来添加。菜单栏由一个文件菜单和这三个动作条目组成。此外我们已经准备了一个文件对话框，它可以让我们选择我们的城市文档。对话框在定义时是不可见的，需要使用```open()```方法来显示它。

```
...
Action {
    id: save
    text: qsTr("&Save")
    shortcut: StandardKey.Save
    onTriggered: { }
}

Action {
    id: open
    text: qsTr("&Open")
    shortcut: StandardKey.Open
    onTriggered: {}
}

Action {
    id: exit
    text: qsTr("E&xit")
    onTriggered: Qt.quit();
}

menuBar: MenuBar {
    Menu {
        title: qsTr("&File")
        MenuItem { action: open }
        MenuItem { action: save }
        MenuSeparator { }
        MenuItem { action: exit }
    }
}

...

FileDialog {
    id: openDialog
    onAccepted: { }
}
```

