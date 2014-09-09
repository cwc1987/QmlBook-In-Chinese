# Web Sockets

webSockets不是Qt提供的。将WebSockets加入到Qt/QML中需要花费一些工作。从作者的角度来看WebSockets有巨大的潜力来添加HTTP服务缺少的功能-通知。HTTP给了我们get和post的功能，但是post还不是一个通知。目前客户端轮询服务器来获得应用程序的服务，服务器也需要能通知客户端变化和事件。你可以与QML接口比较：属性，函数，信号。也可以叫做获取/设置/调用和通知。

QML WebSocket插件将会在Qt5中加入。你可以试试来自qt playground的web sockets插件。为了测试，我们使用一个现有的web socket服务实现了echo server。

首先确保你使用的Qt5.2.x。

```
$ qmake --version
... Using Qt version 5.2.0 ...
```

然后你需要克隆web socket的代码库，并且编译它。

```
$ git clone git@gitorious.org:qtplayground/websockets.git
$ cd websockets
$ qmake
$ make
$ make install
```

现在你可以在qml模块中使用web socket。

```
import Qt.WebSockets 1.0

WebSocket {
    id: socket
}
```

测试你的web socket，我们使用来自[http://websocket.org](http://websocket.org)的echo server 。

```
import QtQuick 2.0
import Qt.WebSockets 1.0

Text {
    width: 480
    height: 48

    horizontalAlignment: Text.AlignHCenter
    verticalAlignment: Text.AlignVCenter

    WebSocket {
        id: socket
        url: "ws://echo.websocket.org"
        active: true
        onTextMessageReceived: {
            text = message
        }
        onStatusChanged: {
            if (socket.status == WebSocket.Error) {
                console.log("Error: " + socket.errorString)
            } else if (socket.status == WebSocket.Open) {
                socket.sendTextMessage("ping")
            } else if (socket.status == WebSocket.Closed) {
                text += "\nSocket closed"
            }
        }
    }
}
```

你可以看到我们使用socket.sendTextMessage("ping")作为响应在文本区域中。

![](http://qmlbook.org/_images/ws_echo.png)

## 11.8.1 WS Server

你可以使用Qt WebSocket的C++部分来创建你自己的WS Server或者使用一个不同的WS实现。它非常有趣，是因为它允许连接使用大量扩展的web应用程序服务的高质量渲染的QML。在这个例子中，我们将使用基于web socket的ws模块的Node JS。你首先需要安装node.js。然后创建一个ws_server文件夹，使用node package manager（npm）安装ws包。

```
$ cd ws_server
$ npm install ws
```

npm工具下载并安装了ws包到你的本地依赖文件夹中。

一个server.js文件是我们服务器的实现。服务器代码将在端口3000创建一个web socket服务并监听连接。在一个连接加入后，它将会发送一个欢迎并等待客户端信息。每个客户端发送到socket信息都会发送回客户端。

```
var WebSocketServer = require('ws').Server;

var server = new WebSocketServer({ port : 3000 });

server.on('connection', function(socket) {
	console.log('client connected');
	socket.on('message', function(msg) {
		console.log('Message: %s', msg);
		socket.send(msg);
	});
	socket.send('Welcome to Awesome Chat');
});

console.log('listening on port ' + server.options.port);
```

你需要获取使用的JavaScript标记和回调函数。

## 11.8.2 WS Client

在客户端我们需要一个链表视图来显示信息，和一个文本输入来输入新的聊天信息。

在例子中我们使用一个白色的标签。

```
// Label.qml
import QtQuick 2.0

Text {
    color: '#fff'
    horizontalAlignment: Text.AlignLeft
    verticalAlignment: Text.AlignVCenter
}
```

我们的聊天视图是一个链表视图，文本被加入到链表模型中。每个条目显示使用行前缀和信息标签。我们使用单元将它分为24列。

```
// ChatView.qml
import QtQuick 2.0

ListView {
    id: root
    width: 100
    height: 62

    model: ListModel {}

    function append(prefix, message) {
        model.append({prefix: prefix, message: message})
    }

    delegate: Row {
        width: root.width
        height: 18
        property real cw: width/24
        Label {
            width: cw*1
            height: parent.height
            text: model.prefix
        }
        Label {
            width: cw*23
            height: parent.height
            text: model.message
        }
    }
}
```

聊天输入框是一个简单的使用颜色包裹边界的文本输入。

```
// ChatInput.qml
import QtQuick 2.0

FocusScope {
    id: root
    width: 240
    height: 32
    Rectangle {
        anchors.fill: parent
        color: '#000'
        border.color: '#fff'
        border.width: 2
    }

    property alias text: input.text

    signal accepted(string text)

    TextInput {
        id: input
        anchors.left: parent.left
        anchors.right: parent.right
        anchors.verticalCenter: parent.verticalCenter
        anchors.leftMargin: 4
        anchors.rightMargin: 4
        onAccepted: root.accepted(text)
        color: '#fff'
        focus: true
    }
}
```

当web socket返回一个信息后，它将会把信息添加到聊天视图中。这也同样适用于状态改变。也可以当用户输入一个聊天信息，将聊天信息拷贝添加到客户端的聊天视图中，并将信息发送给服务器。

```
// ws_client.qml
import QtQuick 2.0
import Qt.WebSockets 1.0

Rectangle {
    width: 360
    height: 360
    color: '#000'

    ChatView {
        id: box
        anchors.left: parent.left
        anchors.right: parent.right
        anchors.top: parent.top
        anchors.bottom: input.top
    }
    ChatInput {
        id: input
        anchors.left: parent.left
        anchors.right: parent.right
        anchors.bottom: parent.bottom
        focus: true
        onAccepted: {
            print('send message: ' + text)
            socket.sendTextMessage(text)
            box.append('>', text)
            text = ''
        }
    }
    WebSocket {
        id: socket

        url: "ws://localhost:3000"
        active: true
        onTextMessageReceived: {
            box.append('<', message)
        }
        onStatusChanged: {
            if (socket.status == WebSocket.Error) {
                box.append('#', 'socket error ' + socket.errorString)
            } else if (socket.status == WebSocket.Open) {
                box.append('#', 'socket open')
            } else if (socket.status == WebSocket.Closed) {
                box.append('#', 'socket closed')
            }
        }
    }
}
```

你首先需要运行服务器，然后是客户端。在我们简单例子中没有客户端重连的机制。

运行服务器

```
$ cd ws_server
$ node server.js
```

运行客户端

```
$ cd ws_client
$ qmlscene ws_client.qml
```

当输入文本并点击发送后，你可以看到类似下面这样。

![](http://qmlbook.org/_images/ws_client.png)
