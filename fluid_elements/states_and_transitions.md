# 状态与过渡（States and Transitions）

通常我们将用户界面描述为一种状态。一个状态定义了一组属性的改变，并且会在一定的条件下被触发。另外在这些状态转化的过程中可以有一个过渡，定义了这些属性的动画或者一些附加的动作。当进入一个新的状态时，动作也可以被执行。

## 5.2.1 状态（States）

在QML中，使用State元素来定义状态，需要与基础元素对象（Item）的states序列属性连接。状态通过它的状态名来鉴别，由组成它的一系列简单的属性来改变元素。默认的状态在初始化元素属性时定义，并命名为“”（一个空的字符串）。

```
    Item {
        id: root
        states: [
            State {
                name: "go"
                PropertyChanges { ... }
            },
            State {
                name: "stop"
                PropertyChanges { ... }
            }
        ]
    }
```

状态的改变由分配一个元素新的状态属性名来完成。

**注意**

**另一种切换属性的方法是使用状态元素的when属性。when属性能够被设置为一个表达式的结果，当结果为true时，状态被使用**。

```
    Item {
        id: root
        states: [
            ...
        ]

        Button {
            id: goButton
            ...
            onClicked: root.state = "go"
        }
    }
```

![](http://qmlbook.org/_images/trafficlight_sketch.png)

例如一个交通信号灯有两个信号灯。上面的一个信号灯使用红色，下面的信号灯使用绿色。在这个例子中，两个信号灯不会同时发光。让我们看看状态图。

![](http://qmlbook.org/_images/trafficlight_states.png)

当系统启动时，它会自动切换到停止模式作为默认状态。停止状态改变了light1为红色并且light2为黑色（关闭）。一个外部的事件能够触发现在的状态变换为“go”状态。在go状态下，我们改变颜色属性，light1变为黑色（关闭），light2变为绿色。

为了实现这个方案，我们给这两个灯绘制一个用户界面的草图，为了简单起见，我们使用两个包含园边的矩形框，设置园半径为宽度的一半（宽度与高度相同）。

```
    Rectangle {
        id: light1
        x: 25; y: 15
        width: 100; height: width
        radius: width/2
        color: "black"
    }

    Rectangle {
        id: light2
        x: 25; y: 135
        width: 100; height: width
        radius: width/2
        color: "black"
    }
```

就像在状态图中定义的一样，我们有一个“go”状态和一个“stop”状态，它们将会分别将交通灯改变为红色和绿色。我们设置state属性到stop来确保初始化状态为stop状态。

**注意**

**我们可以只使用“go”状态来达到同样的效果，设置颜色light1为红色，颜色light2为黑色。初始化状态“”（空字符串）定义初始化属性，并且扮演类似“stop”状态的角色。**

```
    state: "stop"

    states: [
        State {
            name: "stop"
            PropertyChanges { target: light1; color: "red" }
            PropertyChanges { target: light2; color: "black" }
        },
        State {
            name: "go"
            PropertyChanges { target: light1; color: "black" }
            PropertyChanges { target: light2; color: "green" }
        }
    ]
```

PropertyChanges{ target: light2; color: "black" }在这个例子中不是必要的，因为light2初始化颜色已经是黑色了。在一个状态中，只需要描述属性如何从它们的默认状态改变（而不是前一个状态的改变）。

使用鼠标区域覆盖整个交通灯，并且绑定在点击时切换go和stop状态。

```
    MouseArea {
        anchors.fill: parent
        onClicked: parent.state = (parent.state == "stop"? "go" : "stop")
    }
```

![](http://qmlbook.org/_images/trafficlight_ui.png)

我们现在已经成功实现了交通灯的状态切换。为了让用户界面看起来更加自然，我们需要使用动画效果来增加一些过渡。一个过渡能够被状态的改变触发。

**注意**

**可以使用一个简单逻辑的脚本来替换QML状态。开发人员很容易落入这种陷阱，写的代码更像一个JavaScript程序而不是一个QML程序。**

## 5.2.2 过渡（Transitions）

一系列的过渡能够被加入任何元素，一个过渡由状态的改变触发执行。你可以使用属性的from:和to:来定义状态改变的指定过渡。这两个属性就像一个过滤器，当过滤器为true时，过渡生效。你也可以使用“*”来表示任何状态。例如from:"*"; to:"*"表示从任一状态到另一个任一状态的默认值，这意味着过渡用于每个状态的切换。

在这个例子中，我们期望从状态“go”到“stop”转换时实现一个颜色改变的动画。对于从“stop”到“go”状态的改变，我们期望保持颜色的直接改变，不使用过渡。我们使用from和to来限制过渡只在从“go”到“stop”时生效。在过渡中我们给每个灯添加两个颜色的动画，这个动画将按照状态的描述来改变属性。

```
    transitions: [
        Transition {
            from: "stop"; to: "go"
            ColorAnimation { target: light1; properties: "color"; duration: 2000 }
            ColorAnimation { target: light2; properties: "color"; duration: 2000 }
        }
    ]
```

你可以点击用户界面来改变状态。试试点击用户界面，当状态从“stop”到“go”时，你将会发现改变立刻发生了。

![](http://qmlbook.org/_images/trafficlight_transition.png)

接下来，你可以修改下这个例子，例如缩小未点亮的等来突出点亮的等。为此，你需要在状态中添加一个属性用来缩放，并且操作一个动画来播放缩放属性的过渡。另一个选择是可以添加一个“attention”状态，灯会出现黄色闪烁，为此你需要添加为这个过渡添加一个一秒连续的动画来显示黄色（使用“to”属性来实现，一秒后变为黑色）。也许你也可以改变缓冲曲线来使这个例子更加生动。
