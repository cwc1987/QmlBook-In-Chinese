# 高级用法（Advanced Techniques）

## 6.5.1 路径视图（The PathView）

路径视图（PathView）非常强大，但也非常复杂，这个视图由QtQuick提供。它创建了一个可以让子项沿着任意路径移动的视图。沿着相同的路径，使用缩放（scale），透明（opacity）等元素可以更加详细的控制过程。

当使用路径视图（PathView）时，你必须定义一个代理和一个路径。在这些之上，路径视图（PathView）本身也可以自定义一些属性的区间。通常会使用pathItemCount属性，它控制了一次可见的子项总数。preferredHighLightBegin属性控制了高亮区间，preferredHighlightEnd与highlightRangeMode，控制了当前项怎样沿着路径显示。

在关注高亮区间之前，我们必须先看看路径（path）这个属性。路径（path）属性使用一个路径（path）元素来定义路径视图（PathView）内代理的滚动路径。路径使用startx与starty属性来链接路径（path）元素，例如PathLine,PathQuad和PathCubic。这些元素都使用二维数组来构造路径。

当路径定义好之后，可以使用PathPercent和PathAttribute元素来进一步设置。它们被放置在路径元素之间，并且为经过它们的路径和代理提供更加细致的控制。PathPercent提供了如何控制每个元素之间覆盖区域部分的路径，然后反过来控制分布在这条路径上的代理元素，它们被按比例的分布播放。

preferredHightlightBegin与preferredHighlightEnd属性由PathView（路径视图）输入到图片元素中。它们的值在0~1之间。结束值大于等于开始值。例如设置这些属性值为0.5，当前项只会显示当前百分之50的图像在这个路径上。

在Path中，PathAttribute元素也是被放置在元素之间的，就像PathPercent元素。它们可以让你指定属性的值然后插入的路径中去。这些属性与代理绑定可以用来控制任意的属性。

![](http://qmlbook.org/_images/pathview-coverview.png)

下面这个例子展示了路径视图（PathView）如何创建一个卡片视图，并且用户可以滑动它。我们使用了一些技巧来完成这个例子。路径由PathLine元素组成。使用PathPercent元素，它确保了中间的元素居中，并且给其它的元素提供了足够的空间。使用PathAttribute元素来控制旋转，大小和深度值（z-value）。

在这个路径之上（path），需要设置路径视图（PathView）的pathItemCount属性。它控制了路径的浓密度。路径视图的路径（PathView.onPath）使用preferredHighlightBegin与preferredHighlightEnd来控制可见的代理项。

```
    PathView {
        anchors.fill: parent

        delegate: flipCardDelegate
        model: 100

        path: Path {
            startX: root.width/2
            startY: 0

            PathAttribute { name: "itemZ"; value: 0 }
            PathAttribute { name: "itemAngle"; value: -90.0; }
            PathAttribute { name: "itemScale"; value: 0.5; }
            PathLine { x: root.width/2; y: root.height*0.4; }
            PathPercent { value: 0.48; }
            PathLine { x: root.width/2; y: root.height*0.5; }
            PathAttribute { name: "itemAngle"; value: 0.0; }
            PathAttribute { name: "itemScale"; value: 1.0; }
            PathAttribute { name: "itemZ"; value: 100 }
            PathLine { x: root.width/2; y: root.height*0.6; }
            PathPercent { value: 0.52; }
            PathLine { x: root.width/2; y: root.height; }
            PathAttribute { name: "itemAngle"; value: 90.0; }
            PathAttribute { name: "itemScale"; value: 0.5; }
            PathAttribute { name: "itemZ"; value: 0 }
        }

        pathItemCount: 16

        preferredHighlightBegin: 0.5
        preferredHighlightEnd: 0.5
    }
```


代理如下面所示，使用了一些从PathAttribute中链接的属性，itemZ,itemAngle和itemScale。需要注意代理链接的属性只在wrapper中可用。因此，rotxs属性在Rotation元素中定义为可访问值。

另一个需要注意的是路径视图（PathView）链接的PathView.onPath属性的用法。通常对于这个属性都绑定为可见，这样允许路径视图（PathView）缓冲不可见的元素。这不是通过剪裁处理来实现的，因为路径视图（PathView）的代理比其它的视图，例如链表视图（ListView）或者栅格视图（GridView）放置更加随意。

```
    Component {
        id: flipCardDelegate

        Item {
            id: wrapper

            width: 64
            height: 64

            visible: PathView.onPath

            scale: PathView.itemScale
            z: PathView.itemZ

            property variant rotX: PathView.itemAngle
            transform: Rotation { axis { x: 1; y: 0; z: 0 } angle: wrapper.rotX; origin { x: 32; y: 32; } }

            Rectangle {
                anchors.fill: parent
                color: "lightGray"
                border.color: "black"
                border.width: 3
            }

            Text {
                anchors.centerIn: parent
                text: index
                font.pixelSize: 30
            }
        }
    }
```

当在路径视图（PathView）上使用图像转换或者其它更加复杂的元素时，有一个性能优化的技巧是绑定图像元素（Image）的smooth属性与PathView.view.moving属性。这意味着图像在移动时可能不够完美，但是能够比较平滑的转换。当视图在移动时，对于平滑缩放的处理是没有意义的，因为用户根本看不见这个过程。

## 6.5.2 XML模型（A Model from XML）

由于XML是一种常见的数据格式，QML提供了XmlListModel元素来包装XML数据。这个元素能够获取本地或者网络上的XML数据，然后通过XPath解析这些数据。

下面这个例子展示了从RSS流中获取图片，源属性（source）引用了一个网络地址，这个数据会自动下载。

![](http://qmlbook.org/_images/xmllistmodel-images.png)

当数据下载完成后，它会被加工作为模型的子项。查询属性（query）是一个XPath代理的基础查询，用来创建模型项。在这个例子中，这个路径是/rss/channel/item，所以，在一个模型子项创建后，每一个子项的标签，都包含了一个频道标签，包含一个RSS标签。

每一个模型项，一些规则需要被提取，由XmlRole元素来代理。每一个规则都需要一个名称，这样代理才能够通过属性绑定来访问。每个这样的属性的值都通过XPath查询来确定。例如标题属性（title）符合title/string()查询，返回内容中在之间的值。

图像源属性（imageSource）更加有趣，因为它不仅仅是从XML中提取字符串，也需要加载它。在流数据的支持下，每个子项包含了一个图片。使用XPath的函数substring-after与substring-before，可以提取本地的图片资源。这样imageSource属性就可以直接被作为一个Image元素的source属性使用。

```
import QtQuick 2.0
import QtQuick.XmlListModel 2.0

Item {
    width: 300
    height: 480

    Component {
        id: imageDelegate

        Item {
            width: listView.width
            height: 400

	    Column {
                Text {
                    text: title
                }

                Image {
                    source: imageSource
                }
            }
        }
    }

    XmlListModel {
        id: imageModel

        source: "http://feeds.nationalgeographic.com/ng/photography/photo-of-the-day/"
        query: "/rss/channel/item"

        XmlRole { name: "title"; query: "title/string()" }
        XmlRole { name: "imageSource"; query: "substring-before(substring-after(description/string(), 'img src=\"'), '\"')" }
    }

    ListView {
        id: listView

        anchors.fill: parent

        model: imageModel
        delegate: imageDelegate
    }
}
```

## 6.5.3 链表分段（Lists with Sections）

有时，链表的数据需要划分段。例如使用首字母来划分联系人，或者音乐。使用链表视图可以把平面列表按类别划分。

![](http://qmlbook.org/_images/listview-sections.png)

为了使用分段，section.property与section.criteria必须安装。section.property定义了哪些属性用于内容的划分。在这里，最重要的是知道每一段由哪些连续的元素构成，否则相同的属性名可能出现在几个不同的地方。

section.criteria能够被设置为ViewSection.FullString或者ViewSection.FirstCharacter。默认下使用第一个值，能够被用于模型中有清晰的分段，例如音乐专辑。第二个是使用一个属性的首字母来分段，这说明任何属性都可以被使用。通常的例子是用于联系人名单的姓。

当段被定义好后，每个子项能够使用绑定属性ListView.section，ListView.previousSection与ListView.nextSection来访问。使用这些属性，可以检测段的第一个与最后一个子项。

使用链表视图（ListView）的section.delegate属性可以给段指定代理组件。它能够创建段标题，并且可以在任意子项之前插入这个段代理。使用绑定属性section可以访问当前段的名称。

下面这个例子使用国际分类展示了分段的一些概念。国籍（nation）作为section.property，段代理组件（section.delegate）使用每个国家作为标题。在每个段中，spacemen模型中的名字使用spaceManDelegate组件来代理显示。

```
import QtQuick 2.0

Rectangle {
    width: 300
    height: 290

    color: "white"

    ListView {
        anchors.fill: parent
        anchors.margins: 20

        clip: true

        model: spaceMen

        delegate: spaceManDelegate

        section.property: "nation"
        section.delegate: sectionDelegate
    }

    Component {
        id: spaceManDelegate

        Item {
            width: 260
            height: 20

            Text {
                anchors.left: parent.left
                anchors.verticalCenter: parent.verticalCenter
                anchors.leftMargin: 10

                font.pixelSize: 12

                text: name
            }
        }
    }

    Component {
        id: sectionDelegate

        Rectangle {
            width: 260
            height: 20

            color: "lightGray"

            Text {
                anchors.left: parent.left
                anchors.verticalCenter: parent.verticalCenter
                anchors.leftMargin: 10

                font.pixelSize: 12
                font.bold: true

                text: section
            }
        }
    }


    ListModel {
        id: spaceMen

        ListElement { name: "Abdul Ahad Mohmand"; nation: "Afganistan"; }
        ListElement { name: "Marcos Pontes"; nation: "Brazil"; }
        ListElement { name: "Alexandar Panayotov Alexandrov"; nation: "Bulgaria"; }
        ListElement { name: "Georgi Ivanov"; nation: "Bulgaria"; }
        ListElement { name: "Roberta Bondar"; nation: "Canada"; }
        ListElement { name: "Marc Garneau"; nation: "Canada"; }
        ListElement { name: "Chris Hadfield"; nation: "Canada"; }
        ListElement { name: "Guy Laliberte"; nation: "Canada"; }
        ListElement { name: "Steven MacLean"; nation: "Canada"; }
        ListElement { name: "Julie Payette"; nation: "Canada"; }
        ListElement { name: "Robert Thirsk"; nation: "Canada"; }
        ListElement { name: "Bjarni Tryggvason"; nation: "Canada"; }
        ListElement { name: "Dafydd Williams"; nation: "Canada"; }
    }
}
```

## 6.5.4 性能协调（Tunning Performance）

一个模型视图的性能很大程度上依赖于代理的创建。例如滚动下拉一个链表视图时，代理从外部加入到视图底部，并且从视图顶部移出。如果设置剪裁（clip）属性为false，并且代理项花了很多时间来初始化，用户会感觉到视图滚动体验很差。

为了优化这个问题，你可以在滚动时使用像素来调整。使用cacheBuffer属性，在上诉情况下的垂直滚动，它将会调整在链表视图的上下需要预先准备好多少像素的代理项，结合异步加载图像元素（Image），例如在它们进入视图之前加载。

创建更多的代理项将会牺牲一些流畅的体验，并且花更多的时间来初始化每个代理。这并不代表可以解决一些更加复杂的代理项的问题。在每次实例化代理时，它的内容都会被评估和编辑。这需要花费时间，如果它花费了太多的时间，它将会导致一个很差的滚动体验。在一个代理中包含太多的元素也会降低滚动的性能。

为了补救这个问题，我们推荐使用动态加载元素。当它们需要时，可以初始化这些附加的元素。例如，一个展开代理可能推迟它的详细内容的实例化，直到需要使用它时。每个代理中最好减少JavaScript的数量。将每个代理中复杂的JavaScript调用放在外面来实现。这将会减少每个代理在创建时编译JavaScript。
