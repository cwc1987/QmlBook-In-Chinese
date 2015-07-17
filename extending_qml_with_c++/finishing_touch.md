# 收尾工作（Finishing Touch）

这个应用程序还没有真正的完成。我们想要显示旗帜，并允许用户通过从数据模型中移除城市来修改文档。

这些旗帜被存放在```main.qml```文件夹下的```flags```文件夹中。为了在表格列中显示它们，我们需要定义一个渲染旗帜图片的代理。

```
TableViewColumn {
    delegate: Item {
        Image {
            anchors.centerIn: parent
            source: 'flags/' + styleData.value
        }
    }
    role: 'flag'
    title: "Flag"
    width: 40
}
```

它将JS数据模型中暴露的flag属性作为```styleData.value```交给代理。代理调整图片路径，并在路径前面加上```'flags/'```并显示它。

对于移除，我们使用相似的技巧来显示一个移除按钮。

```
TableViewColumn {
    delegate: Button {
        iconSource: "remove.png"
        onClicked: {
            var data = view.model
            data.splice(styleData.row, 1)
            view.model = data
        }
    }
    width: 40
}
```

数据移除操作，我们坚持从视图模型上获取数据，然后使用JS的```splice```函数移除一个条目。这个方法提供给我们的模型来自一个JS数组。```splice```方法通过移除已有元素，添加新的元素来改变数组内容。

一个JS数组不如一个Qt模型智能，例如```QAbstractItemModel```，它无法通知视图行更新或者数据更新。由于视图无法接收到任何更新的通知，它无法更新数据显示。只有在我们将数据重新设置回视图时，视图才会知道有新的数据需要刷新视图内容。使用```view.model = data```再次设置数据模型可以让视图知道有数据更新。

![](http://qmlbook.github.io/_images/cityui_populated.png)
