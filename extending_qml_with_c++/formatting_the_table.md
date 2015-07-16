# 格式化表格（Formatting the Table）

城市数据的内容应该被现实在一个表格中。我们使用```TableView```
控制并定义4列：城市，国家，面积，人口。每一列都是典型的```TableViewColumn```。然后我们添加列的标识并移除要求自定义列代理的操作。

```
TableView {
    id: view
    anchors.fill: parent
    TableViewColumn {
        role: 'city'
        title: "City"
        width: 120
    }
    TableViewColumn {
        role: 'country'
        title: "Country"
        width: 120
    }
    TableViewColumn {
        role: 'area'
        title: "Area"
        width: 80
    }
    TableViewColumn {
        role: 'population'
        title: "Population"
        width: 80
    }
}
```

现在应用程序能够显示一个包含文件菜单的菜单栏和一个包含4个表头的空表格。下一步是我们的```FileIO```扩展将有用的数据填充到表格中。

![](http://qmlbook.github.io/_images/cityui_empty.png)

文档cities.json是一组城市条目。这里是一个例子。

```
[
    {
        "area": "1928",
        "city": "Shanghai",
        "country": "China",
        "flag": "22px-Flag_of_the_People's_Republic_of_China.svg.png",
        "population": "13831900"
    },
    ...
]
```

我们任务是允许用户选择文件，读取它，转换它，并将它设置到表格视图中。
