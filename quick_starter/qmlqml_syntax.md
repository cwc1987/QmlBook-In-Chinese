# QML语法（QML Syntax）

QML是一种描述用户界面的声明式语言。它将用户界面分解成一些更小的元素，这些元素能够结合成一个组件。QML语言描述了用户界面元素的形状和行为。用户界面能够使用JavaScript来提供修饰，或者增加更加复杂的逻辑。从这个角度来看它遵循HTML-JavaScript模式，但QML是被设计用来描述用户界面的，而不是文本文档。

从QML元素的层次结构来理解是最简单的学习方式。子元素从父元素上继承了坐标系统，它的x,y坐标总是相对应于它的父元素坐标系统。

![](http://qmlbook.org/_images/scene.png)

让我们开始用一个简单的QML文件例子来解释这个语法。

```
// rectangle.qml

import QtQuick 2.0

// The root element is the Rectangle
Rectangle {
    // name this element root
    id: root

    // properties: <name>: <value>
    width: 120; height: 240

    // color property
    color: "#D8D8D8"

    // Declare a nested element (child of root)
    Image {
    	id: rocket

        // reference the parent
        x: (parent.width - width)/2; y: 40

        source: 'assets/rocket.png'
    }

    // Another child of root
    Text {
        // un-named element

        // reference element by id
        y: rocket.y + rocket.height + 20

        // reference root element
        width: root.width

        horizontalAlignment: Text.AlignHCenter
        text: 'Rocket'
    }
}
```

* import声明导入了一个指定的模块版本。一般来说会导入QtQuick2.0来作为初始元素的引用。

* 使用//可以单行注释，使用/**/可以多行注释，就像C/C++和JavaScript一样。

* 每一个QML文件都需要一个根元素，就像HTML一样。

* 一个元素使用它的类型声明，然后使用{}进行包含。

* 元素拥有属性，他们按照name:value的格式来赋值。

* 任何在QML文档中的元素都可以使用它们的id进行访问（id是一个任意的标识符）。

* 元素可以嵌套，这意味着一个父元素可以拥有多个子元素。子元素可以通过访问parent关键字来访问它们的父元素。

**建议**

**你会经常使用id或者关键字parent来访问你的父对象。有一个比较好的方法是命名你的根元素对象id为root（id:root），这样就不用去思考你的QML文档中的根元素应该用什么方式命名了。**

**提示**

**你可以在你的操作系统命令行模式下使用QtQuick运行环境来运行这个例子，比如像下面这样：**

```
$ $QTDIR/bin/qmlscene rectangle.qml
```

将$QTDIR替换为你的Qt的安装路径。qmlscene会执行Qt Quick运行环境初始化，并且解释这个QML文件。

在Qt Creator中你可以打开对应的项目文件然后运行rectangle.qml文档。

## 4.1.1 属性（Properties）

元素使用他们的元素类型名进行声明，使用它们的属性或者创建自定义属性来定义。一个属性对应一个值，例如 width:100，text: 'Greeting', color: '#FF0000'。一个属性有一个类型定义并且需要一个初始值。

```
    Text {
        // (1) identifier
        id: thisLabel

        // (2) set x- and y-position
        x: 24; y: 16

        // (3) bind height to 2 * width
        height: 2 * width

        // (4) custom property
        property int times: 24

        // (5) property alias
        property alias anotherTimes: thisLabel.times

        // (6) set text appended by value
        text: "Greetings " + times

        // (7) font is a grouped property
        font.family: "Ubuntu"
        font.pixelSize: 24

        // (8) KeyNavigation is an attached property
        KeyNavigation.tab: otherLabel

        // (9) signal handler for property changes
        onHeightChanged: console.log('height:', height)

        // focus is neeed to receive key events
        focus: true

        // change color based on focus value
        color: focus?"red":"black"
    }
```

让我们来看看不同属性的特点：

1. id是一个非常特殊的属性值，它在一个QML文件中被用来引用元素。id不是一个字符串，而是一个标识符和QML语法的一部分。一个id在一个QML文档中是唯一的，并且不能被设置为其它值，也无法被查询（它的行为更像C++世界里的指针）。

2. 一个属性能够设置一个值，这个值依赖于它的类型。如果没有对一个属性赋值，那么它将会被初始化为一个默认值。你可以查看特定的元素的文档来获得这些初始值的信息。

3. 一个属性能够依赖一个或多个其它的属性，这种操作称作属性绑定。当它依赖的属性改变时，它的值也会更新。这就像订了一个协议，在这个例子中height始终是width的两倍。

4. 添加自己定义的属性需要使用property修饰符，然后跟上类型，名字和可选择的初始化值（property <type> <name> : <value>）。如果没有初始值将会给定一个系统初始值作为初始值。**注意如果属性名与已定义的默认属性名不重复，使用default关键字你可以将一个属性定义为默认属性。这在你添加子元素时用得着，如果他们是可视化的元素，子元素会自动的添加默认属性的子类型链表（children property list）**。

5. 另一个重要的声明属性的方法是使用alias关键字（property alias <name> : <reference>）。alias关键字允许我们转发一个属性或者转发一个属性对象自身到另一个作用域。我们将在后面定义组件导出内部属性或者引用根级元素id会使用到这个技术。一个属性别名不需要类型，它使用引用的属性类型或者对象类型。

6. text属性依赖于自定义的timers（int整型数据类型）属性。int整型数据会自动的转换为string字符串类型数据。这样的表达方式本身也是另一种属性绑定的例子，文本结果会在times属性每次改变时刷新。

7. 一些属性是按组分配的属性。当一个属性需要结构化并且相关的属性需要联系在一起时，我们可以这样使用它。另一个组属性的编码方式是 font{family: "UBuntu"; pixelSize: 24 }。

8. 一些属性是元素自身的附加属性。这样做是为了全局的相关元素在应用程序中只出现一次（例如键盘输入）。编码方式<element>.<property>: <value> 。

9. 对于每个元素你都可以提供一个信号操作。这个操作在属性值改变时被调用。例如这里我们完成了当height（高度）改变时会使用控制台输出一个信息。

**警告**

**一个元素id应该只在当前文档中被引用。QML提供了动态作用域的机制，后加载的文档会覆盖之前加载文档的元素id号，这样就可以引用已加载并且没有被覆盖的元素id，这有点类似创建全局变量。但不幸的是这样的代码阅读性很差。目前这个还没有办法解决这个问题，所以你使用这个机制的时候最好仔细一些甚至不要使用这种机制。如果你想向文档外提供元素的调用，你可以在根元素上使用属性导出的方式来提供。**

## 4.1.2 脚本（Scripting）

QML与JavaScript是最好的配合。在JavaScrpit的章节中我们将会更加详细的介绍这种关系，现在我们只需要了解这种关系就可以了。

```
    Text {
        id: label

        x: 24; y: 24

        // custom counter property for space presses
        property int spacePresses: 0

        text: "Space pressed: " + spacePresses + " times"

        // (1) handler for text changes
        onTextChanged: console.log("text changed to:", text)

        // need focus to receive key events
        focus: true

        // (2) handler with some JS
        Keys.onSpacePressed: {
            increment()
        }

        // clear the text on escape
        Keys.onEscapePressed: {
            label.text = ''
        }

        // (3) a JS function
        function increment() {
            spacePresses = spacePresses + 1
        }
    }
```

1. 文本改变操作onTextChanged会将每次空格键按下导致的文本改变输出到控制台。

2. 当文本元素接收到空格键操作（用户在键盘上点击空格键），会调用JavaScript函数increment()。

3. 定义一个JavaScript函数使用这种格式function (){....}，在这个例子中是增加spacePressed的计数。每次spacePressed的增加都会导致它绑定的属性更新。

**注意**

**QML的：（属性绑定）与JavaScript的=（赋值）是不同的。绑定是一个协议，并且存在于整个生命周期。然而JavaScript赋值（=）只会产生一次效果。当一个新的绑定生效或者使用JavaScript赋值给属性时，绑定的生命周期就会结束。例如一个按键的操作设置文本属性为一个空的字符串将会销毁我们的增值显示：**

```
Keys.onEscapePressed: {
    label.text = ''
}
```

**在点击取消（ESC）后，再次点击空格键（space-bar）将不会更新我们的显示，之前的text属性绑定（text: "Space pressed:" + spacePresses + "times")被销毁。**

**当你对改变属性的策略有冲突时（文本的改变基于一个增值的绑定并且可以被JavaScript赋值清零），类似于这个例子，你最好不要使用绑定属性。你需要使用赋值的方式来改变属性，属性绑定会在赋值操作后被销毁（销毁协议！）。**

