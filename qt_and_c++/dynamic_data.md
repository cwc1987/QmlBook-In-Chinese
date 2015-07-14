# 动态数据（Dynamic Data）

动态数据包含了从模型中插入，移除，清除数据等。```QAbstractListModel```期望当条目被移除或者插入时有一个明确的行为。这个行为使用一个信号来表示，在操作调用前和调用后调用这个行为。例如向一个模型插入一行数据，你首先需要发送```beginInsertRows```信号，然后操作数据，最后发送```endInsertRows```信号。

我们将在头文件中加入后续的函数。这些使用```Q_INVOKABLE```函数定义使得可以在QML中调用它们。另一种方法是将它们定义为公共槽函数。

```
// inserts a color at the index (0 at begining, count-1 at end)
Q_INVOKABLE void insert(int index, const QString& colorValue);
// uses insert to insert a color at the end
Q_INVOKABLE void append(const QString& colorValue);
// removes a color from the index
Q_INVOKABLE void remove(int index);
// clear the whole model (e.g. reset)
Q_INVOKABLE void clear();
```

此外，我们定义了```count```属性来获取模型的大小和一个使用索引值的```get```方法来获取颜色。这些东西在QML中使用迭代器遍历模型时会用到。

```
// gives the size of the model
Q_PROPERTY(int count READ count NOTIFY countChanged)
// gets a color at the index
Q_INVOKABLE QColor get(int index);
```

实现插入数据首先要检查边界和插入值是否有效。在这之后我们开始插入数据。

```
void DynamicEntryModel::insert(int index, const QString &colorValue)
{
    if(index < 0 || index > m_data.count()) {
        return;
    }
    QColor color(colorValue);
    if(!color.isValid()) {
        return;
    }
    // view protocol (begin => manipulate => end]
    emit beginInsertRows(QModelIndex(), index, index);
    m_data.insert(index, color);
    emit endInsertRows();
    // update our count property
    emit countChanged(m_data.count());
}
```

添加数据非常简单。我们使用模型大小并调用插入函数来实现。

```
void DynamicEntryModel::append(const QString &colorValue)
{
    insert(count(), colorValue);
}
```

移除数据与插入数据类似，但是需要调用移除操作协议。

```
void DynamicEntryModel::remove(int index)
{
    if(index < 0 || index >= m_data.count()) {
        return;
    }
    emit beginRemoveRows(QModelIndex(), index, index);
    m_data.removeAt(index);
    emit endRemoveRows();
    // do not forget to update our count property
    emit countChanged(m_data.count());
}
```

函数```count```不太重要，这里不再介绍，只需要知道它会返回数据总数。```get```函数也十分简单。

```
QColor DynamicEntryModel::get(int index)
{
    if(index < 0 || index >= m_data.count()) {
        return QColor();
    }
    return m_data.at(index);
}
```

你需要注意你只能返回一个QML可读取的值。如果它不是QML基础类型或者QML所知类型，你需要使用qmlRegisterType或者qmlRegisterUncreatableType注册类型。如果是用户不能在QML中实例化对象的类型应该使用qmlRegisterUncreatableType注册。

现在你可以在QML中使用模型并且可以从模型中插入，添加，移除条目。这里有一个小例子，它允许用户输入一个颜色名称或者颜色16进制值，并将这个颜色加入到模型中在链表视图中显示。代理上的红色圆圈允许用户从模型中移除这个条目。在条目被移除后，模型通知链表视图更新它的内容。

![](http://qmlbook.github.io/_images/modelview.png)

这里是QML代码。你可以在这章的资源里找到完整的源代码。这个例子使用了QtQuick.Controls和QtQuick.Layout模块使得代码更加紧凑。控制模块提供了QtQuick中一组与桌面相关的用户界面元素，布局模块提供了非常有用的布局管理器。

```
import QtQuick 2.2
import QtQuick.Window 2.0
import QtQuick.Controls 1.2
import QtQuick.Layouts 1.1

// our module
import org.example 1.0

Window {
    visible: true
    width: 480
    height: 480


    Background { // a dark background
        id: background
    }

    // our dyanmic model
    DynamicEntryModel {
        id: dynamic
        onCountChanged: {
            // we print out count and the last entry when count is changing
            print('new count: ' + count);
            print('last entry: ' + get(count-1));
        }
    }

    ColumnLayout {
        anchors.fill: parent
        anchors.margins: 8
        ScrollView {
            Layout.fillHeight: true
            Layout.fillWidth: true
            ListView {
                id: view
                // set our dynamic model to the views model property
                model: dynamic
                delegate: ListDelegate {
                    width: ListView.view.width
                    // construct a string based on the models proeprties
                    text: 'hsv(' +
                          Number(model.hue).toFixed(2) + ',' +
                          Number(model.saturation).toFixed() + ',' +
                          Number(model.brightness).toFixed() + ')'
                    // sets the font color of our custom delegates
                    color: model.name

                    onClicked: {
                        // make this delegate the current item
                        view.currentIndex = index
                        view.focus = true
                    }
                    onRemove: {
                        // remove the current entry from the model
                        dynamic.remove(index)
                    }
                }
                highlight: ListHighlight { }
                // some fun with transitions :-)
                add: Transition {
                    // applied when entry is added
                    NumberAnimation {
                        properties: "x"; from: -view.width;
                        duration: 250; easing.type: Easing.InCirc
                    }
                    NumberAnimation { properties: "y"; from: view.height;
                        duration: 250; easing.type: Easing.InCirc
                    }
                }
                remove: Transition {
                    // applied when entry is removed
                    NumberAnimation {
                        properties: "x"; to: view.width;
                        duration: 250; easing.type: Easing.InBounce
                    }
                }
                displaced: Transition {
                    // applied when entry is moved
                    // (e.g because another element was removed)
                    SequentialAnimation {
                        // wait until remove has finished
                        PauseAnimation { duration: 250 }
                        NumberAnimation { properties: "y"; duration: 75
                        }
                    }
                }
            }
        }
        TextEntry {
            id: textEntry
            onAppend: {
                // called when the user presses return on the text field
                // or clicks the add button
                dynamic.append(color)
            }

            onUp: {
                // called when the user presses up while the text field is focused
                view.decrementCurrentIndex()
            }
            onDown: {
                // same for down
                view.incrementCurrentIndex()
            }

        }
    }
}
```

模型-视图编程是Qt中最难的任务之一。对于正常的应用开发者，它是为数不多的需要实现接口的类。其它类你只需要正常使用就可以额。模型的草图通常在QML这边开始。你需要想象你的用户在QML中需要什么样的模型。通常建议创建协议时首先使用ListModel看看如何在QML中更好的工作。这种方法对于定义QML编程接口同样有效。使数据从C++到QML中可用不仅仅是技术边界，也是从命令式编程到声明式编程的编程方法转变。所以准备好经历一些挫折并从中获取快乐吧。
