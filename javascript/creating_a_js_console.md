# 创建JS控制台（Creating a JS Console）

下面这个小的例子我们将创建一个JS控制台。我们需要一个输入区域允许用户输入表达式，和一个结果输出列表。由于这更像一个桌面应用程序，我们使用QtQuick控制模块。

**注意**

**在你下一个项目中包含一个JS控制台可以更好的用来测试。增加Quake-Terminal效果也有助于对你的客户留下更好的印象。为了更好的使用它，你需要评估JS控制台的控制范围，例如当前可见屏幕，核心数据模型，一个单例对象或者其它的东西。**

![](http://qmlbook.github.io/_images/jsconsole.png)

我们在Qt Creator中使用QtQuick controls创建一个Qt Quick UI项目。把这个项目取名为JSConsole。完成引导后，我们已经有了一个基础的应用程序框架，这个框架包含一个应用程序窗口和一个菜单。

我们使用一个TextFiled来输入文本，使用一个Button来对输入求值。表达式求值结果使用一个机遇链表模型（ListModel）的链表视图（ListView）显示，每一个链表项使用两个标签显示表达式和求值结果。

```
// part of JSConsole.qml
ApplicationWindow {
  id: root

  ...

  ColumnLayout {
      anchors.fill: parent
      anchors.margins: 9
      RowLayout {
          Layout.fillWidth: true
          TextField {
              id: input
              Layout.fillWidth: true
              focus: true
              onAccepted: {
                  // call our evaluation function on root
                  root.jsCall(input.text)
              }
          }
          Button {
              text: qsTr("Send")
              onClicked: {
                  // call our evaluation function on root
                  root.jsCall(input.text)
              }
          }
      }
      Item {
          Layout.fillWidth: true
          Layout.fillHeight: true
          Rectangle {
              anchors.fill: parent
              color: '#333'
              border.color: Qt.darker(color)
              opacity: 0.2
              radius: 2
          }

          ScrollView {
              id: scrollView
              anchors.fill: parent
              anchors.margins: 9
              ListView {
                  id: resultView
                  model: ListModel {
                      id: outputModel
                  }
                  delegate: ColumnLayout {
                      width: ListView.view.width
                      Label {
                          Layout.fillWidth: true
                          color: 'green'
                          text: "> " + model.expression
                      }
                      Label {
                          Layout.fillWidth: true
                          color: 'blue'
                          text: "" + model.result
                      }
                      Rectangle {
                          height: 1
                          Layout.fillWidth: true
                          color: '#333'
                          opacity: 0.2
                      }
                  }
              }
          }
      }
  }
}
```

求值函数jsCall不会做求值操作，而是将它的内容移动到JS模块（jsconsole.js）完成清晰的分离。

```
// part of JSConsole.qml

import "jsconsole.js" as Util

...

ApplicationWindow {
  id: root

  ...

  function jsCall(exp) {
      var data = Util.call(exp);
      // insert the result at the beginning of the list
      outputModel.insert(0, data)
  }
}
```

为了安全我们不在JS中调用eval函数，这可能导致用户修改局部作用域。我们使用constructor函数在运行时创建JS函数，并在我们的作用域中传入变量值。由于函数可能随时都在创建，它不能扮演关闭和存储它私有作用域的角色，我们需要使用 this.a = 10 来存储值10在这个函数的作用域。这个作用域由脚本设置到变量作用域。

```
// jsconsole.js
.pragma library

var scope = {
  // our custom scope injected into our function evaluation
}

function call(msg) {
    var exp = msg.toString();
    console.log(exp)
    var data = {
        expression : msg
    }
    try {
        var fun = new Function('return (' + exp + ');');
        data.result = JSON.stringify(fun.call(scope), null, 2)
        console.log('scope: ' + JSON.stringify(scope, null, 2) + 'result: ' + result)
    } catch(e) {
        console.log(e.toString())
        data.error = e.toString();
    }
    return data;
}
```

调用函数返回的数据是一个JS对象，包含了一个结果，表达式和错误属性：data: { expression: {}, result: {}, error: {} }。我们可以直接在链表模型（ListModel）中使用这个JS对象，并且可以通过代理（delegate）访问它，例如model.expression会得到输入的表达式。为了让例子更加简单，我们忽略了错误的结果。


