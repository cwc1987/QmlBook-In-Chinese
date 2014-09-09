# 代理（Delegate）

当使用模型与视图来自定义用户界面时，代理在创建显示时扮演了大量的角色。在模型中的每个元素通过代理来实现可视化，用户真实可见的是这些代理元素。

每个代理访问到索引号或者绑定的属性，一些是来自数据模型，一些来自视图。来自模型的数据将会通过属性传递到代理。来自视图的数据将会通过属性传递视图中与代理相关的状态信息。

通常使用的视图绑定属性是ListView.isCurrentItem和ListView.view。第一个是一个布尔值，标识这个元素是否是视图当前元素，这个值是只读的，引用自当前视图。通过访问视图，可以创建可复用的代理，这些代理在被包含时会自动匹配视图的大小。在下面这个例子中，每个代理的width（宽度）属性与视图的width（宽度）属性绑定，每个代理的背景颜色color依赖于绑定的属性ListView.isCurrentItem属性。

```
import QtQuick 2.0

Rectangle {
    width: 120
    height: 300

    color: "white"

    ListView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: 100

        delegate: numberDelegate
        spacing: 5

        focus: true
    }

    Component {
        id: numberDelegate

        Rectangle {
            width: ListView.view.width
            height: 40

            color: ListView.isCurrentItem?"gray":"lightGray"

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: index
            }
        }
    }
}
```

![](http://qmlbook.org/_images/delegates-basic.png)

如果在模型中的每个元素与一个动作相关，例如点击作用于一个元素时，这个功能是代理完成的。这是由事件管理分配给视图的，这个操作控制了视图中元素的导航，代理控制了特定元素上的动作。

最基础的方法是在每个代理中创建一个MouseArea（鼠标区域）并且响应onClicked信号。在后面章节中将会演示这个例子。

## 6.4.1 动画添加与移除元素（Animating Added and Removed Items）

在某些情况下，视图中的显示内容会随着时间而改变。由于模型数据的改变，元素会添加或者移除。在这些情况下，一个比较好的做法是使用可视化队列给用户一个方向的感觉来帮助用户知道哪些数据被加入或者移除。

为了方便使用，QML视图为每个代理绑定了两个信号，onAdd和onRemove。使用动画连接它们，可以方便创建识别哪些内容被添加或删除的动画。

下面这个例子演示了如何动态填充一个链表模型（ListModel）。在屏幕下方，有一个添加新元素的按钮。当点击它时，会调用模型的append方法来添加一个新的元素。这个操作会触发视图创建一个新的代理，并发送GridView.onAdd信号。SequentialAnimation队列动画与这个信号连接绑定，使用代理的scale属性来放大视图元素。

当视图中的一个代理点击时，将会调用模型的remove方法将一个元素从模型中移除。这个操作将会导致GridView.onRemove信号的发送，触发另一个SequentialAnimation。这时，代理的销毁将会延迟直到动画完成。为了完成这个操作，PropertyAction元素需要在动画前设置GridView.delayRemove属性为true，并在动画后设置为false。这样确保了动画在代理项移除前完成。

```
import QtQuick 2.0

Rectangle {
    width: 480
    height: 300

    color: "white"

    ListModel {
        id: theModel

        ListElement { number: 0 }
        ListElement { number: 1 }
        ListElement { number: 2 }
        ListElement { number: 3 }
        ListElement { number: 4 }
        ListElement { number: 5 }
        ListElement { number: 6 }
        ListElement { number: 7 }
        ListElement { number: 8 }
        ListElement { number: 9 }
    }

    Rectangle {
        anchors.left: parent.left
        anchors.right: parent.right
        anchors.bottom: parent.bottom
        anchors.margins: 20

        height: 40

        color: "darkGreen"

        Text {
            anchors.centerIn: parent

            text: "Add item!"
        }

        MouseArea {
            anchors.fill: parent

            onClicked: {
                theModel.append({"number": ++parent.count});
            }
        }

        property int count: 9
    }

    GridView {
        anchors.fill: parent
        anchors.margins: 20
        anchors.bottomMargin: 80

        clip: true

        model: theModel

        cellWidth: 45
        cellHeight: 45

        delegate: numberDelegate
    }

    Component {
        id: numberDelegate

        Rectangle {
            id: wrapper

            width: 40
            height: 40

            color: "lightGreen"

            Text {
                anchors.centerIn: parent

                font.pixelSize: 10

                text: number
            }

            MouseArea {
                anchors.fill: parent

                onClicked: {
                    if (!wrapper.GridView.delayRemove)
                        theModel.remove(index);
                }
            }

            GridView.onRemove: SequentialAnimation {
                PropertyAction { target: wrapper; property: "GridView.delayRemove"; value: true }
                NumberAnimation { target: wrapper; property: "scale"; to: 0; duration: 250; easing.type: Easing.InOutQuad }
                PropertyAction { target: wrapper; property: "GridView.delayRemove"; value: false }
            }

            GridView.onAdd: SequentialAnimation {
                NumberAnimation { target: wrapper; property: "scale"; from: 0; to: 1; duration: 250; easing.type: Easing.InOutQuad }
            }
        }
    }
}
```

## 6.4.2 形变的代理（Shape-Shifting Delegates）

在使用链表时通常会使用当前项激活时展开的机制。这个操作可以被用于动态的将当前项目填充到整个屏幕来添加一个新的用户界面，或者为链表中的当前项提供更多的信息。

在下面的例子中，当点击链表项时，链表项都会展开填充整个链表视图（ListView）。额外的间隔区域被用于添加更多的信息，这种机制使用一个状态来控制，当一个链表项展开时，代理项都能输入expanded（展开）状态，在这种状态下一些属性被改变。

首先，包装器（wrapper）的高度（height）被设置为链表视图（ListView）的高度。标签图片被放大并且下移，使图片从小图片的位置移向大图片的位置。除了这些之外，两个隐藏项，实际视图（factsView）与关闭按键（closeButton）切换它的opactiy（透明度）显示出来。最后设置链表视图（ListView）。

设置链表视图（ListView）包含了设置内容Y坐标（contentsY），这是视图顶部可见的部分代理的Y轴坐标。另一个变化是设置视图的交互（interactive）为false。这个操作阻止了视图的移动，用户不再能够通过滚动条切换当前项。

由于设置第一个链表项为可点击，向它输入一个expanded（展开）状态，导致了它的代理项被填充到整个链表并且内容重置。当点击关闭按钮时，清空状态，导致它的代理项返回上一个状态，并且重新设置链表视图（ListView）有效。

```
import QtQuick 2.0

Item {
    width: 300
    height: 480

    ListView {
        id: listView

        anchors.fill: parent

        delegate: detailsDelegate
        model: planets
    }

    ListModel {
        id: planets

        ListElement { name: "Mercury"; imageSource: "images/mercury.jpeg"; facts: "Mercury is the smallest planet in the Solar System. It is the closest planet to the sun. It makes one trip around the Sun once every 87.969 days." }
        ListElement { name: "Venus"; imageSource: "images/venus.jpeg"; facts: "Venus is the second planet from the Sun. It is a terrestrial planet because it has a solid, rocky surface. The other terrestrial planets are Mercury, Earth and Mars. Astronomers have known Venus for thousands of years." }
        ListElement { name: "Earth"; imageSource: "images/earth.jpeg"; facts: "The Earth is the third planet from the Sun. It is one of the four terrestrial planets in our Solar System. This means most of its mass is solid. The other three are Mercury, Venus and Mars. The Earth is also called the Blue Planet, 'Planet Earth', and 'Terra'." }
        ListElement { name: "Mars"; imageSource: "images/mars.jpeg"; facts: "Mars is the fourth planet from the Sun in the Solar System. Mars is dry, rocky and cold. It is home to the largest volcano in the Solar System. Mars is named after the mythological Roman god of war because it is a red planet, which signifies the colour of blood." }
    }

    Component {
        id: detailsDelegate

        Item {
            id: wrapper

            width: listView.width
            height: 30

            Rectangle {
                anchors.left: parent.left
                anchors.right: parent.right
                anchors.top: parent.top

                height: 30

                color: "#ffaa00"

                Text {
                    anchors.left: parent.left
                    anchors.verticalCenter: parent.verticalCenter

                    font.pixelSize: parent.height-4

                    text: name
                }
            }

            Rectangle {
                id: image

                color: "black"

                anchors.right: parent.right
                anchors.top: parent.top
                anchors.rightMargin: 2
                anchors.topMargin: 2

                width: 26
                height: 26

                Image {
                    anchors.fill: parent

                    fillMode: Image.PreserveAspectFit

                    source: imageSource
                }
            }

            MouseArea {
                anchors.fill: parent
                onClicked: parent.state = "expanded"
            }

            Item {
                id: factsView

                anchors.top: image.bottom
                anchors.left: parent.left
                anchors.right: parent.right
                anchors.bottom: parent.bottom

                opacity: 0

                Rectangle {
                    anchors.fill: parent

                    color: "#cccccc"

                    Text {
                        anchors.fill: parent
                        anchors.margins: 5

                        clip: true
                        wrapMode: Text.WordWrap

                        font.pixelSize: 12

                        text: facts
                    }
                }
            }

            Rectangle {
                id: closeButton

                anchors.right: parent.right
                anchors.top: parent.top
                anchors.rightMargin: 2
                anchors.topMargin: 2

                width: 26
                height: 26

                color: "red"

                opacity: 0

                MouseArea {
                    anchors.fill: parent
                    onClicked: wrapper.state = ""
                }
            }

            states: [
                State {
                    name: "expanded"

                    PropertyChanges { target: wrapper; height: listView.height }
                    PropertyChanges { target: image; width: listView.width; height: listView.width; anchors.rightMargin: 0; anchors.topMargin: 30 }
                    PropertyChanges { target: factsView; opacity: 1 }
                    PropertyChanges { target: closeButton; opacity: 1 }
                    PropertyChanges { target: wrapper.ListView.view; contentY: wrapper.y; interactive: false }
                }
            ]

            transitions: [
                Transition {
                    NumberAnimation {
                        duration: 200;
                        properties: "height,width,anchors.rightMargin,anchors.topMargin,opacity,contentY"
                    }
                }
            ]
        }
    }
}
```

![](http://qmlbook.org/_images/delegates-expanding-small.png)

![](http://qmlbook.org/_images/delegates-expanding-large.png)

这个技术展示了展开代理来填充视图能够简单的通过代理的形变来完成。例如当浏览一个歌曲的链表时，可以通过放大当前项来对该项添加更多的说明。
