# 基础模型（Basic Model）

最基本的分离数据与显示的方法是使用Repeater元素。它被用于实例化一组元素项，并且很容易与一个用于填充用户界面的定位器相结合。

最基本的实现举例，repeater元素用于实现子元素的标号。每个子元素都拥有一个可以访问的属性index，用于区分不同的子元素。在下面的例子中，一个repeater元素创建了10个子项，子项的数量由model属性控制。对于每个子项Rectangle包含了一个Text元素，你可以将text属性设置为index的值，因此可以看到子项的编号是0~9。

```
import QtQuick 2.0

Column {
    spacing: 2

    Repeater {
        model: 10

        Rectangle {
            width: 100
            height: 20

            radius: 3

            color: "lightBlue"

            Text {
                anchors.centerIn: parent
                text: index
            }
        }
    }
}
```

![](http://qmlbook.org/_images/repeater-number.png)

这是一个不错的编号列表，有时我们想显示一些更复杂的数据。使用一个JavaScript序列来替换整形变量model的值可以达到我们的目的。序列可以使用任何类型的内容，可以是字符串，整数，或者对象。在下面的例子中，使用了一个字符串链表。我们仍然使用index的值作为变量，并且我们也访问modelData中包含的每个元素的数据。

```
import QtQuick 2.0

Column {
    spacing: 2

    Repeater {
        model: ["Enterprise", "Colombia", "Challenger", "Discovery", "Endeavour", "Atlantis"]

        Rectangle {
            width: 100
            height: 20

            radius: 3

            color: "lightBlue"

            Text {
                anchors.centerIn: parent
                text: index +": "+modelData
            }
        }
    }
}
```

![](http://qmlbook.org/_images/repeater-array.png)

将数据暴露成一组序列，你可以通过标号迅速的找到你需要的信息。想象一下这个模型的草图，这是一个最简单的模型，也是通常都会使用的模型，ListModel（链表模型）。一个链表模型由许多ListElement（链表元素）组成。在每个链表元素中，可以绑定值到属性上。例如在下面这个例子中，每个元素都提供了一个名字和一个颜色。

每个元素中的属性绑定连接到repeater实例化的子项上。这意味着变量name和surfaceColor可以被repeater创建的每个Rectangle和Text项引用。这不仅可以方便的访问数据，也可以使源代码更加容易阅读。surfaceColor是名字左边圆的颜色，而不是模糊的数据序列列i或者行j。

```
import QtQuick 2.0

Column {
    spacing: 2

    Repeater {
        model: ListModel {
            ListElement { name: "Mercury"; surfaceColor: "gray" }
            ListElement { name: "Venus"; surfaceColor: "yellow" }
            ListElement { name: "Earth"; surfaceColor: "blue" }
            ListElement { name: "Mars"; surfaceColor: "orange" }
            ListElement { name: "Jupiter"; surfaceColor: "orange" }
            ListElement { name: "Saturn"; surfaceColor: "yellow" }
            ListElement { name: "Uranus"; surfaceColor: "lightBlue" }
            ListElement { name: "Neptune"; surfaceColor: "lightBlue" }
        }

        Rectangle {
            width: 100
            height: 20

            radius: 3

            color: "lightBlue"

            Text {
                anchors.centerIn: parent
                text: name
            }

            Rectangle {
                anchors.left: parent.left
                anchors.verticalCenter: parent.verticalCenter
                anchors.leftMargin: 2

                width: 16
                height: 16

                radius: 8

                border.color: "black"
                border.width: 1

                color: surfaceColor
            }
        }
    }
}
```

![](http://qmlbook.org/_images/repeater-model.png)

repeater的内容的每个子项实例化时绑定了默认的属性delegate（代理）。这意味着例1（第一个代码段）的代码与下面显示的代码是相同的。注意，唯一的不同是delegate属性名，将会在后面详细讲解。

```
import QtQuick 2.0

Column {
    spacing: 2

    Repeater {
        model: 10

        delegate: Rectangle {
            width: 100
            height: 20

            radius: 3

            color: "lightBlue"

            Text {
                anchors.centerIn: parent
                text: index
            }
        }
    }
}
```
