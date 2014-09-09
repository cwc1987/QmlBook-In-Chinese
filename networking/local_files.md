# 本地文件（Local files）

使用XMLHttpRequest也可以加载本地文件（XML/JSON）。例如加载一个本地名为“colors.json”的文件可以这样使用：

```
xhr.open("GET", "colors.json");
```

我们使用它读取一个颜色表并且使用表格来显示。从QtQuick这边无法修改文件。为了将源数据存储回去，我们需要一个基于HTTP服务器的REST服务支持或者一个用来访问文件的QtQuick扩展。

```
import QtQuick 2.0

Rectangle {
    width: 360
    height: 360
    color: '#000'

    GridView {
        id: view
        anchors.fill: parent
        cellWidth: width/4
        cellHeight: cellWidth
        delegate: Rectangle {
            width: view.cellWidth
            height: view.cellHeight
            color: modelData.value
        }
    }

    function request() {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.HEADERS_RECEIVED) {
                print('HEADERS_RECEIVED')
            } else if(xhr.readyState === XMLHttpRequest.DONE) {
                print('DONE');
                var obj = JSON.parse(xhr.responseText.toString());
                view.model = obj.colors
            }
        }
        xhr.open("GET", "colors.json");
        xhr.send();
    }

    Component.onCompleted: {
        request()
    }
}
```

也可以使用XmlListModel来替代XMLHttpRequest访问本地文件。

```
import QtQuick.XmlListModel 2.0

XmlListModel {
    source: "http://localhost:8080/colors.xml"
    query: "/colors"
    XmlRole { name: 'color'; query: 'name/string()' }
    XmlRole { name: 'value'; query: 'value/string()' }
}
```

XmlListModel只能用来读取XML文件，不能读取JSON文件。
