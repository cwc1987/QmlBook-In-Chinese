# 使用FileIO（Using FileIO）

现在我们可以使用新创建的文件访问一些简单的数据。这个例子中我们想要读取一个JSON格式下的城市数据并且在表格中显示。我们将使用两个项目，一个是扩展插件项目（叫做```fileio```），提供读取和写入文件的方法。另外一个项目通过fileio读取/写入文件将数据显示在表格中（```CityUI```）。这个例子中使用的数据在```cities.json```文件中。

![](http://qmlbook.github.io/_images/cityui_mock.png)

JSON只是文本，它被格式化为可以转换为一个有效的JS对象/数组并返回一个文本。我们使用```FileIO```读取格式化的JSON数据并使用```JSON.parse()```将它转换为一个JS对象。数据在后面被用作一个表格视图的数据模型。我们粗略的阅读函数文档就可以获取这些内容。为了保存数据我们将转换回文本格式并使用写入函数保存。

城市的JSON数据是一个格式化文本文件，包含了一组城市数据条目，每个条目包含了关于城市数据。

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
