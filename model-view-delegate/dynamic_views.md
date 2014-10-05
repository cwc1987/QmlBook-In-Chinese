# 动态视图（Dynamic Views）

Repeater元素适合有限的静态数据，但是在真正使用时，模型通常更加复杂和庞大，我们需要一个更加智能的解决方案。QtQuick提供了ListView和GridView元素，这两个都是基于Flickable（可滑动）区域的元素，因此用户可以放入更大的数据。同时，它们限制了同时实例化的代理数量。对于一个大型的模型，这意味着在同一个场景下只会加载有限的元素。

![](http://qmlbook.org/_images/listview-basic.png)

![](http://qmlbook.org/_images/gridview-basic.png)

这两个元素的用法非常类似，我们由ListView开始，然后会描述GridView的模型起点来进行比较。

ListView与Repeater元素像素，它使用了一个model，使用delegate来实例化，并且在两个delegate之间能够设置间隔sapcing。下面的列表显示了怎样设置一个简单的链表。

```
import QtQuick 2.0

Rectangle {
    width: 80
    height: 300

    color: "white"

    ListView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: 100

        delegate: numberDelegate
        spacing: 5
    }

    Component {
        id: numberDelegate

        Rectangle {
            width: 40
            height: 40

            color: "lightGreen"

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: index
            }
        }
    }
}
```

![](http://qmlbook.org/_images/listview-basic.png)

如果模型包含的数据比屏幕上显示的更多，ListView元素只会显示部分的链表内容。然后由于QtQuick的默认行为导致的问题，列表视图不会限制被显示的代理项（delegates）只在限制区域内显示。这意味着代理项可以在列表视图外显示，用户可以看见在列表视图外动态的创建和销毁这些代理项（delegates）。为了防止这个问题，ListView通过设置clip属性为true，来激活裁剪功能。下面的图片展示了这个结果，左边是clip属性设置为false的对比。

![](http://qmlbook.org/_images/listview-clip.png)

对于用户，ListView（列表视图）是一个滚动区域。它支持惯性滚动，这意味着它可以快速的翻阅内容。默认模式下，它可以在内容最后继续伸展，然后反弹回去，这个信号告诉用户已经到达内容的末尾。

视图末尾的行为是由到boundsBehavior属性的控制的。这是一个枚举值，并且可以配置为默认的Flickable.DragAndOvershootBounds，视图可以通过它的边界线来拖拽和翻阅，配置为Flickable.StopAtBounds，视图将不再可以移动到它的边界线之外。配置为Flickable.DragOverBounds，用户可以将视图拖拽到它的边界线外，但是在边界线上翻阅将无效。

使用snapMode属性可以限制一个视图内元素的停止位置。默认行为下是ListView.NoSnap，允许视图内元素在任何位置停止。将snapMode属性设置为ListView.SnapToItem，视图顶部将会与元素对象的顶部对齐排列。使用ListView.SnapOneItem，当鼠标或者触摸释放时，视图将会停止在第一个可见的元素，这种模式对于浏览页面非常便利。

## 6.3.1 方向（Orientation）

默认的链表视图只提供了一个垂直方向的滚动条，但是水平滚动条也是需要的。链表视图的方向由属性orientation控制。它能够被设置为默认值ListView.Vertical或者ListView.Horizontal。下面是一个水平链表视图。

```
import QtQuick 2.0

Rectangle {
    width: 480
    height: 80

    color: "white"

    ListView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: 100

        orientation: ListView.Horizontal

        delegate: numberDelegate
        spacing: 5
    }

    Component {
        id: numberDelegate

        Rectangle {
            width: 40
            height: 40

            color: "lightGreen"

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: index
            }
        }
    }
}
```

![](http://qmlbook.org/_images/listview-horizontal.png)

按照上面的设置，水平链表视图默认的元素顺序方向是由左到右。可以通过设置layoutDirection属性来控制元素顺序方向，它可以设置为Qt.LeftToRight或者Qt.RightToLeft。

## 6.3.2 键盘导航和高亮

当使用基于触摸方式的链表视图时，默认提供的视图已经足够使用。在使用键盘甚至仅仅通过方向键选择一个元素的场景下，需要有标识当前选中元素的机制。在QML中，这被叫做高亮。

视图支持设置一个当前视图中显示代理元素中的高亮代理。它是一个附加的代理元素，这个元素仅仅只实例化一次，并移动到与当前元素相同的位置。

在下面例子的演示中，有两个属性来完成这个工作。首先是focus属性设置为true，它设置链表视图能够获得键盘焦点。然后是highlight属性，指出使用的高亮代理元素。高亮代理元素的x,y与height属性由当前元素指定。如果宽度没有特别指定，当前元素的宽度也可以用于高亮代理元素。

在例子中，ListView.view.width属性被绑定用于高亮元素的宽度。关于代理元素的使绑定属性将在后面的章节讨论，但是最好知道相同的绑定属性也可以用于高亮代理元素。

```
import QtQuick 2.0

Rectangle {
    width: 240
    height: 300

    color: "white"

    ListView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: 100

        delegate: numberDelegate
        spacing: 5

        highlight: highlightComponent
        focus: true
    }

    Component {
        id: highlightComponent

        Rectangle {
            width: ListView.view.width
            color: "lightGreen"
        }
    }

    Component {
        id: numberDelegate

        Item {
            width: 40
            height: 40

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: index
            }
        }
    }
}
// M1>>
```

![](http://qmlbook.org/_images/listview-highlight.png)

当使用高亮与链表视图（ListView）结合时，一些属性可以用来控制它的行为。highlightRangeMode控制了高亮如何影响视图中当前的显示。默认设置ListView.NoHighLighRange意味着高亮与视图中的元素距离不相关。

ListView.StrictlyEnforceRnage确保了高亮始终可见，如果某个动作尝试将高亮移出当前视图可见范围，当前元素将会自动切换，确保了高亮始终可见。

ListView.ApplyRange，它尝试保持高亮代理始终可见，但是不会强制切换当前元素始终可见。如果在需要的情况下高亮代理允许被移出当前视图。

在默认配置下，视图负责高亮移动到指定位置，移动的速度与大小的改变能够被控制，使用一个速度值或者一个动作持续时间来完成它。这些属性包括highlightMoveSpeed，highlightMoveDuration，highlightResizeSpeed和highlightResizeDuration。默认下速度被设置为每秒400像素，动作持续时间为-1，表明速度和距离控制了动作的持续时间。如果速度与动作持续时间都被设置，动画将会采用速度较快的结果来完成。

为了更加详细的控制高亮的移动，highlightFollowCurrentItem属性设置为false。这意味着视图将不再负责高亮代理的移动。取而代之可以通过一个行为（Bahavior）或者一个动画来控制它。

在下面的例子中，高亮代理的y坐标属性与ListView.view.currentItem.y属性绑定。这确保了高亮始终跟随当前元素。然而，由于我们没有让视图来移动这个高亮代理，我们需要控制这个元素如何移动，通过Behavior on y来完成这个操作，在下面的例子中，移动分为三步完成：淡出，移动，淡入。注意怎样使用SequentialAnimation和PropertyAnimation元素与NumberAnimation结合创建更加复杂的移动效果。

```
    Component {
        id: highlightComponent

        Item {
            width: ListView.view.width
            height: ListView.view.currentItem.height

            y: ListView.view.currentItem.y

            Behavior on y {
                SequentialAnimation {
                    PropertyAnimation { target: highlightRectangle; property: "opacity"; to: 0; duration: 200 }
                    NumberAnimation { duration: 1 }
                    PropertyAnimation { target: highlightRectangle; property: "opacity"; to: 1; duration: 200 }
                }
            }

            Rectangle {
                id: highlightRectangle
                anchors.fill: parent
                color: "lightGreen"
            }
        }
    }
```

## 6.3.3 页眉与页脚（Header and Footer）

这一节是链表视图最后的内容，我们能够向链表视图中插入一个页眉（header）元素和一个页脚（footer）元素。这部分是链表的开始或者结尾处被作为代理元素特殊的区域。对于一个水平链表视图，不会存在页眉或者页脚，但是也有开始和结尾处，这取决于layoutDirection的设置。

下面这个例子展示了如何使用一个页眉和页脚来突出链表的开始与结尾。这些特殊的链表元素也有其它的作用，例如，它们能够保持链表中的按键加载更多的内容。

```
import QtQuick 2.0

Rectangle {
    width: 80
    height: 300

    color: "white"

    ListView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: 4

        delegate: numberDelegate
        spacing: 5

        header: headerComponent
        footer: footerComponent
    }

    Component {
        id: headerComponent

        Rectangle {
            width: 40
            height: 20

            color: "yellow"
        }
    }

    Component {
        id: footerComponent

        Rectangle {
            width: 40
            height: 20

            color: "red"
        }
    }

    Component {
        id: numberDelegate

        Rectangle {
            width: 40
            height: 40

            color: "lightGreen"

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: index
            }
        }
    }
}
```

**注意**

**页眉与页脚代理元素不遵循链表视图（ListView）的间隔（spacing）属性，它们被直接放在相邻的链表元素之上或之下。这意味着页眉与页脚的间隔必须通过页眉与页脚元素自己设置。**

![](http://qmlbook.org/_images/listview-header-footer.png)

## 6.3.4 网格视图（The GridView）

使用网格视图（GridView）与使用链表视图（ListView）的方式非常类似。真正不同的地方是网格视图（GridView）使用了一个二维数组来存放元素，而链表视图（ListView）是使用的线性链表来存放元素。

![](http://qmlbook.org/_images/gridview-basic.png)

与链表视图（ListView）比较，网格视图（GridView）不依赖于元素间隔和大小来配置元素。它使用单元宽度（cellWidth）与单元高度（cellHeight）属性来控制数组内的二维元素的内容。每个元素从左上角开始依次放入单元格。

```
import QtQuick 2.0

Rectangle {
    width: 240
    height: 300

    color: "white"

    GridView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: 100

        cellWidth: 45
        cellHeight: 45

        delegate: numberDelegate
    }

    Component {
        id: numberDelegate

        Rectangle {
            width: 40
            height: 40

            color: "lightGreen"

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: index
            }
        }
    }
}
```

一个网格视图（GridView）也包含了页脚与页眉，也可以使用高亮代理并且支持捕捉模式（snap mode）的多种反弹行为。它也可以使用不同的方向（orientations）与定向（directions）来定位。

定向使用flow属性来控制。它可以被设置为GridView.LeftToRight或者GridView.TopToBottom。模型的值从左往右向网格中填充，行添加是从上往下。视图使用一个垂直方向的滚动条。后面添加的元素也是由上到下，由左到右。

此外还有flow属性和layoutDirection属性，能够适配网格从左到右或者从右到左，这依赖于你使用的设置值。
