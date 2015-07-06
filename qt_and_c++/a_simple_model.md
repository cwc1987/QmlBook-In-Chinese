# 一个简单的模型（A simple model）

一个典型的QML C++模型继承自```QAbstractListModel```
，并且最少需要实现```data```和```rowCount```函数。在这个例子中我们将使用由```QColor```类提供的一系列SVG颜色名称并且使用我们的模型展示它们。数据被存储在```QList<QString>```数据容器中。

我们的```DataEntryModel```基础自```QAbstractListModel```并且实现了需要强制实现的函数。我们可以在```rowCount```中忽略父对象索引，这只在树模型中使用。```QModelIndex```类提供了视图检索数据需要的单元格行和列的信息，视图基于行列和数据角色从模型中拉取数据。```QAbstractListModel```在```QtCore```中定义，但是```QColor```被定义在```QtGui```中。我们需要附加```QtGui```依赖。对于QML应用程序，它可以依赖```QtGui```，但是它通常不依赖```QtWidgets```。

```
#ifndef DATAENTRYMODEL_H
#define DATAENTRYMODEL_H

#include <QtCore>
#include <QtGui>

class DataEntryModel : public QAbstractListModel
{
    Q_OBJECT
public:
    explicit DataEntryModel(QObject *parent = 0);
    ~DataEntryModel();

public: // QAbstractItemModel interface
    virtual int rowCount(const QModelIndex &parent) const;
    virtual QVariant data(const QModelIndex &index, int role) const;
private:
    QList<QString> m_data;
};

#endif // DATAENTRYMODEL_H
```

现在你可以使用QML导入命令```import org.example 1.0```来访问```DataEntryModel```，和其它QML项使用的方法一样```DataEntryModel {}```。

我们在这个例子中使用它来显示一个简单的颜色条目列表。

```
import org.example 1.0

ListView {
    id: view
    anchors.fill: parent
    model: DataEntryModel {}
    delegate: ListDelegate {
        // use the defined model role "display"
        text: model.display
    }
    highlight: ListHighlight { }
}
```

ListDelegate是自定义用来显示文本的代理。ListHighlight是一个矩形框。保持例子的整洁在代码提取时进行了保留。

视图现在可以使用C++模型来显示字符串列表，并且显示模型的属性。它仍然非常简单，但是已经可以在QML中使用。通常数据由外部的模型提供，这里的模型只是扮演了视图的一个接口。
