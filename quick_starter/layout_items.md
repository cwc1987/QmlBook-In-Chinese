# 布局元素（Layout Items）

QML使用anchors（锚）对元素进行布局。anchoring（锚定）是基础元素对象的基本属性，可以被所有的可视化QML元素使用。一个anchors（锚）就像一个协议，并且比几何变化更加强大。Anchors（锚）是相对关系的表达式，你通常需要与其它元素搭配使用。

![](http://qmlbook.org/_images/anchors.png)

一个元素有6条锚定线（top顶，bottom底，left左，right右，horizontalCenter水平中，verticalCenter垂直中）。在文本元素（Text Element）中有一条文本的锚定基线（baseline）。每一条锚定线都有一个偏移（offset）值，在top（顶），bottom（底），left（左），right（右）的锚定线中它们也被称作边距。对于horizontalCenter（水平中）与verticalCenter（垂直中）与baseline（文本基线）中被称作偏移值。

![](http://qmlbook.org/_images/anchorgrid.png)

1. 元素填充它的父元素。
```
        GreenSquare {
            BlueSquare {
                width: 12
                anchors.fill: parent
                anchors.margins: 8
                text: '(1)'
            }
        }
```

2. 元素左对齐它的父元素。
```
        GreenSquare {
            BlueSquare {
                width: 48
                y: 8
                anchors.left: parent.left
                anchors.leftMargin: 8
                text: '(2)'
            }
        }
```

3. 元素的左边与它父元素的右边对齐。
```
        GreenSquare {
            BlueSquare {
                width: 48
                anchors.left: parent.right
                text: '(3)'
            }
        }
```

4. 元素中间对齐。Blue1与它的父元素水平中间对齐。Blue2与Blue1中间对齐，并且它的顶部对齐Blue1的底部。
```
        GreenSquare {
            BlueSquare {
                id: blue1
                width: 48; height: 24
                y: 8
                anchors.horizontalCenter: parent.horizontalCenter
            }
            BlueSquare {
                id: blue2
                width: 72; height: 24
                anchors.top: blue1.bottom
                anchors.topMargin: 4
                anchors.horizontalCenter: blue1.horizontalCenter
                text: '(4)'
            }
        }
```

5. 元素在它的父元素中居中。
```
        GreenSquare {
            BlueSquare {
                width: 48
                anchors.centerIn: parent
                text: '(5)'
            }
        }
```

6. 元素水平方向居中对齐父元素并向后偏移12像素，垂直方向居中对齐。
```
        GreenSquare {
            BlueSquare {
                width: 48
                anchors.horizontalCenter: parent.horizontalCenter
                anchors.horizontalCenterOffset: -12
                anchors.verticalCenter: parent.verticalCenter
                text: '(6)'
            }
        }
```

**注意**

**我们的方格都打开了拖拽。试着拖放几个方格。你可以发现第一个方格无法被拖拽因为它每个边都被固定了，当然第一个方格的父元素能够被拖拽是因为它的父元素没有被固定。第二个方格能够在垂直方向上拖拽是因为它只有左边被固定了。类似的第三个和第四个方格也只能在垂直方向上拖拽是因为它们都使用水平居中对齐。第五个方格使用居中布局，它也无法被移动，第六个方格与第五个方格类似。拖拽一个元素意味着会改变它的x,y坐标。anchoring（锚定）比几何变化（例如x,y坐标变化）更强大是因为锚定线（anchored lines）的限制，我们将在后面讨论动画时看到这些功能的强大。**
