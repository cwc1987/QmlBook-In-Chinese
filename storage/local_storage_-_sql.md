# 本地存储 - SQL（Local Storage - SQL）

Qt Quick支持一个与浏览器由区别的本地存储编程接口。需要使用"import QtQuick.LocalStorage 2.0"语句来导入后才能使用这个编程接口。

通常使用基于给定的数据库名称和版本号使用系统特定位置的唯一文件ID号来存储数据到一个SQLITE数据库中。无法列出或者删除已有的数据库。你可以使用QQmlEngine::offlineStoragePate()来寻找本地存储。

使用这个编程接口你首选需要创建一个数据库对象，然后在这个数据库上创建数据库事务。每个事务可以包含一个或多个SQL查询。当一个SQL查询在事务中失败后，事务会回滚。

例如你可以使用本地存储从一个简单的注释表中读取一个文本列：

```
import QtQuick 2.2
import QtQuick.LocalStorage 2.0

Item {
    Component.onCompleted: {
        var db = LocalStorage.openDatabaseSync("MyExample", "1.0", "Example database", 10000);
        db.transaction( function(tx) {
            var result = tx.executeSql('select * from notes');
            for(var i = 0; i < result.rows.length; i++) {
                    print(result.rows[i].text);
                }
            }
        });
    }
}
```

## 疯狂的矩形框（Crazy Rectangle）

假设我们想要存储一个矩形在场景中的位置。

![](http://qmlbook.github.io/_images/crazy_rect.png)

下面是我们的基础代码。

```
import QtQuick 2.2

Item {
    width: 400
    height: 400

    Rectangle {
        id: crazy
        objectName: 'crazy'
        width: 100
        height: 100
        x: 50
        y: 50
        color: "#53d769"
        border.color: Qt.lighter(color, 1.1)
        Text {
            anchors.centerIn: parent
            text: Math.round(parent.x) + '/' + Math.round(parent.y)
        }
        MouseArea {
            anchors.fill: parent
            drag.target: parent
        }
    }
}
```

可以自由的拖动这个矩形。当关闭这个应用程序并再次打开时，这个矩形框仍然在相同的位置。

现在我们将添加矩形的x/y坐标值存储到SQL数据库中。首先我们需要添加一个初始化、读取和保存数据库功能。这些功能在组件构造和组件销毁时被调用。

```
import QtQuick 2.2
import QtQuick.LocalStorage 2.0

Item {
    // reference to the database object
    property var db;

    function initDatabase() {
        // initialize the database object
    }

    function storeData() {
        // stores data to DB
    }

    function readData() {
        // reads and applies data from DB
    }


    Component.onCompleted: {
        initDatabase();
        readData();
    }

    Component.onDestruction: {
        storeData();
    }
}
```

你也可以调用已有的JS库提取相关数据库代码来完成所有的逻辑。如果这个逻辑变得更加复杂这将是最好的解决方案。

在数据库初始化函数中，我们创建一个数据库对象并且确保SQL表已经被创建。

```
function initDatabase() {
    print('initDatabase()')
    db = LocalStorage.openDatabaseSync("CrazyBox", "1.0", "A box who remembers its position", 100000);
    db.transaction( function(tx) {
        print('... create table')
        tx.executeSql('CREATE TABLE IF NOT EXISTS data(name TEXT, value TEXT)');
    });
}
```

应用程序下一步调用读取函数来读取数据库中已有的数据。这里我们需要区分数据库表中是否已有数据。我们观察有多少条语句返回来检查是否已有数据。

```
function readData() {
    print('readData()')
    if(!db) { return; }
    db.transaction( function(tx) {
        print('... read crazy object')
        var result = tx.executeSql('select * from data where name="crazy"');
        if(result.rows.length === 1) {
            print('... update crazy geometry')
            // get the value column
            var value = result.rows[0].value;
            // convert to JS object
            var obj = JSON.parse(value)
            // apply to object
            crazy.x = obj.x;
            crazy.y = obj.y;
        }
    });
}
```

我们希望在将数据作为一个JSON字符串存储在一列值中。这与典型的SQL不同，但是可以很好的与JS代码结合。所以我们将它存储为一个JS对象的可以使用JSON stringif/parse方法的数据替代使用x,y作为属性值放在数据库表中。最后我们获取一个包含x,y属性有效的JS对象，我们可以将它应用在我们疯狂的矩形中。

为了保存数据，我们需要区分更新和插入的情况。当一个记录已经存在时我们使用更新，如果没有记录则将它插入在“crazy”下。

```
function storeData() {
    print('storeData()')
    if(!db) { return; }
    db.transaction( function(tx) {
        print('... check if a crazy object exists')
        var result = tx.executeSql('SELECT * from data where name = "crazy"');
        // prepare object to be stored as JSON
        var obj = { x: crazy.x, y: crazy.y };
        if(result.rows.length === 1) {// use update
            print('... crazy exists, update it')
            result = tx.executeSql('UPDATE data set value=? where name="crazy"', [JSON.stringify(obj)]);
        } else { // use insert
            print('... crazy does not exists, create it')
            result = tx.executeSql('INSERT INTO data VALUES (?,?)', ['crazy', JSON.stringify(obj)]);
        }
    });
}
```

替代选择所有记录的设置，我们也可以使用SQLITE计数函数：SELECT COUNT(*) from data where name = "crazy"，将返回使用行选择查询的结果。否则这将是一个通用的SQL代码。所谓额外的特性，我们在查询中使用？绑定SQL值。

现在你可与拖动这个矩形框当你退出应用程序时会将x/y坐标值存储到数据库，下次启动应用程序时矩形框将使用存储的x/y坐标值定位。
