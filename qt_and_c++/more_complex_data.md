# 更复杂的数据（More Complex Data）

实际工作中使用的模型数据通常比较复杂。所以需要自定义一些角色枚举方便视图通过属性查找数据。例如模型提供颜色数据不仅只是16进制字符串，在QML中也可以是来自HSV颜色模型的色调，饱和度和亮度，以“model.hue”，“model.saturation”和“model.brightness”作为参数。

```
#ifndef ROLEENTRYMODEL_H
#define ROLEENTRYMODEL_H

#include <QtCore>
#include <QtGui>

class RoleEntryModel : public QAbstractListModel
{
    Q_OBJECT
public:
    // Define the role names to be used
    enum RoleNames {
        NameRole = Qt::UserRole,
        HueRole = Qt::UserRole+2,
        SaturationRole = Qt::UserRole+3,
        BrightnessRole = Qt::UserRole+4
    };

    explicit RoleEntryModel(QObject *parent = 0);
    ~RoleEntryModel();

    // QAbstractItemModel interface
public:
    virtual int rowCount(const QModelIndex &parent) const override;
    virtual QVariant data(const QModelIndex &index, int role) const override;
protected:
    // return the roles mapping to be used by QML
    virtual QHash<int, QByteArray> roleNames() const override;
private:
    QList<QColor> m_data;
    QHash<int, QByteArray> m_roleNames;
};

#endif // ROLEENTRYMODEL_H
```

在头文件中，我们为QML添加了数据角色枚举的映射。当QML尝试访问一个模型中的属性时（例如“model.name”），链表视图将会在映射中查询“name”然后向模型申请使用```NameRole```角色枚举的数据。用户在定义角色枚举时应该从```Qt::UserRole```开始，并且对于每个模型需要保证唯一。

```
#include "roleentrymodel.h"

RoleEntryModel::RoleEntryModel(QObject *parent)
    : QAbstractListModel(parent)
{
    // Set names to the role name hash container (QHash<int, QByteArray>)
    // model.name, model.hue, model.saturation, model.brightness
    m_roleNames[NameRole] = "name";
    m_roleNames[HueRole] = "hue";
    m_roleNames[SaturationRole] = "saturation";
    m_roleNames[BrightnessRole] = "brightness";

    // Append the color names as QColor to the data list (QList<QColor>)
    for(const QString& name : QColor::colorNames()) {
        m_data.append(QColor(name));
    }

}

RoleEntryModel::~RoleEntryModel()
{
}

int RoleEntryModel::rowCount(const QModelIndex &parent) const
{
    Q_UNUSED(parent);
    return m_data.count();
}

QVariant RoleEntryModel::data(const QModelIndex &index, int role) const
{
    int row = index.row();
    if(row < 0 || row >= m_data.count()) {
        return QVariant();
    }
    const QColor& color = m_data.at(row);
    qDebug() << row << role << color;
    switch(role) {
    case NameRole:
        // return the color name as hex string (model.name)
        return color.name();
    case HueRole:
        // return the hue of the color (model.hue)
        return color.hueF();
    case SaturationRole:
        // return the saturation of the color (model.saturation)
        return color.saturationF();
    case BrightnessRole:
        // return the brightness of the color (model.brightness)
        return color.lightnessF();
    }
    return QVariant();
}

QHash<int, QByteArray> RoleEntryModel::roleNames() const
{
    return m_roleNames;
}
```

现在实现只是改变了两个地方。首先是初始化。我们使用```QColor```数据类型初始化数据链表。此外我们还定义了我们自己的角色名称映射实现QML的访问。这个映射将在后面的```::roleNames```函数中返回。

第二个变化是在```::data```函数中。我们确保能够覆盖到其它的角色枚举（例如色调，饱和度，亮度）。没有可以从颜色中获取SVG名称的方法，由于一个颜色可以替代任何颜色，但SVG名称是受限的。所以我们忽略掉这点。我们需要创建一个结构体```{ QColor, QString }```来存储名称，这样可以鉴别已被命名的颜色。

在注册类型完成后，我们可以使用模型了，可以将它的条目显示在我们的用户界面中。

```
ListView {
    id: view
    anchors.fill: parent
    model: RoleEntryModel {}
    focus: true
    delegate: ListDelegate {
        text: 'hsv(' +
              Number(model.hue).toFixed(2) + ',' +
              Number(model.saturation).toFixed() + ',' +
              Number(model.brightness).toFixed() + ')'
        color: model.name
    }
    highlight: ListHighlight { }
}
```

我们将返回的类型转换为JS数字类型，这样可以使用定点标记来格式化数字。代码中应当避免直接调用数字（例如```model.saturation.toFixed(2)```）。选择哪种格式取决于你的输入数据。

