# REST接口（REST API）

为了使用web服务，我们首先需要创建它。我们使用Flask（http://flask.pocoo.org），一个基于python创建简单的颜色web服务的HTTP服务器应用。你也可以使用其它的web服务器，只要它接收和返回JSON数据。通过web服务来管理一组已经命名的颜色。在这个例子中，管理意味着CRUD（创建-读取-更新-删除）。

在Flask中一个简单的web服务可以写入一个文件。我们使用一个空的服务器.py文件开始，在这个文件中我们创建一些规则并且从额外的JSON文件中加载初始颜色。你可以查看Flask文档获取更多的帮助。

```
from flask import Flask, jsonify, request
import json

colors = json.load(file('colors.json', 'r'))

app = Flask(__name__)

# ... service calls go here

if __name__ == '__main__':
    app.run(debug = True)
```

当你运行这个脚本后，它会在http://localhost:5000。

我们开始添加我们的CRUD（创建，读取，更新，删除）到我们的web服务。

## 11.5.1 读取请求（Read Request）

从web服务读取数据，我们提供GET方法来读取所有的颜色。

```
@app.route('/colors', methods = ['GET'])
def get_colors():
    return jsonify( { "colors" :  colors })
```

这将会返回‘/colors’下的颜色。我们使用curl来创建一个http请求测试。

```
curl -i -GET http://localhost:5000/colors
```

这将会返回给我们JSON数据的颜色链表。

## 11.5.2 读取接口（Read Entry）

为了通过名字读取颜色，我们提供更加详细的后缀，定位在‘/colors/'下。名称是后缀的参数，用来识别一个独立的颜色。

```
@app.route('/colors/<name>', methods = ['GET'])
def get_color(name):
    for color in colors:
        if color["name"] == name:
            return jsonify( color )
```

我们再次使用curl测试，例如获取一个红色的接口。

```
curl -i -GET http://localhost:5000/colors/red
```

这将返回一个JSON数据的颜色。

## 11.5.3 创建接口（Create Entry）

目前我们仅仅使用了HTTP GET方法。为了在服务器端创建一个接口，我们使用POST方法，并且将新的颜色信息发使用POST数据发送。后缀与获取所有颜色相同，但是我们需要使用一个POST请求。

```
@app.route('/colors', methods= ['POST'])
def create_color():
    color = {
        'name': request.json['name'],
        'value': request.json['value']
    }
    colors.append(color)
    return jsonify( color ), 201
```

curl非常灵活，允许我们使用JSON数据作为新的接口包含在POST请求中。

```
curl -i -H "Content-Type: application/json" -X POST -d '{"name":"gray1","value":"#333"}' http://localhost:5000/colors
```

## 11.5.4 更新接口（Update Entry）

我们使用PUT HTTP方法来添加新的update接口。后缀与取得一个颜色接口相同。当颜色更新后，我们获取更新后JSON数据的颜色。

```
@app.route('/colors/<name>', methods= ['PUT'])
def update_color(name):
    for color in colors:
        if color["name"] == name:
            color['value'] = request.json.get('value', color['value'])
            return jsonify( color )
```

在curl请求中，我们用JSON数据来定义更新值，后缀名用来识别哪个颜色需要更新。

```
curl -i -H "Content-Type: application/json" -X PUT -d '{"value":"#666"}' http://localhost:5000/colors/red
```

## 11.5.5 删除接口（Delete Entry）

使用DELETE HTTP来完成删除接口。使用与颜色相同的后缀，但是使用DELETE HTTP方法。

```
@app.route('/colors/<name>', methods=['DELETE'])
def delete_color(name):
    success = False
    for color in colors:
        if color["name"] == name:
            colors.remove(color)
            success = True
    return jsonify( { 'result' : success } )
```

这个请求看起来与GET请求一个颜色类似。

```
curl -i -X DELETE http://localhost:5000/colors/red
```

现在我们能够读取所有颜色，读取指定颜色，创建新的颜色，更新颜色和删除颜色。我们知道使用HTTP后缀来访问我们的接口。

| 动作 | HTTP协议 | 后缀  |
| -- | -- | -- |
| 读取所有 | GET | http://localhost:5000/colors   |
| 创建接口 | POST | http://localhost:5000/colors  |
| 读取接口 | GET | http://localhost:5000/colors/name |
| 更新接口 | PUT | http://localhost:5000/colors/name |
| 删除接口 | DELETE  | http://localhost:500/colors/name |

REST服务已经完成，我们现在只需要关注QML和客户端。为了创建一个简单好用的接口，我们需要映射每个动作为一个独立的HTTP请求，并且给我们的用户提供一个简单的接口。

## 11.5.6 REST客户端（REST Client）

为了展示REST客户端，我们写了一个小的颜色表格。这个颜色表格显示了通过HTTP请求从web服务取得的颜色。我们的用户界面提供以下命令：

* 获取颜色链表

* 创建颜色

* 读取最后的颜色

* 更新最后的颜色

* 删除最后的颜色

我们将我们的接口包装在一个JS文件中，叫做colorservice.js，并将它导入到我们的UI中作为服务（Service）。在服务模块中，我们创建了帮助函数来为我们构造HTTP请求：

```
// colorservice.js
function request(verb, endpoint, obj, cb) {
    print('request: ' + verb + ' ' + BASE + (endpoint?'/' + endpoint:''))
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        print('xhr: on ready state change: ' + xhr.readyState)
        if(xhr.readyState === XMLHttpRequest.DONE) {
            if(cb) {
                var res = JSON.parse(xhr.responseText.toString())
                cb(res);
            }
        }
    }
    xhr.open(verb, BASE + (endpoint?'/' + endpoint:''));
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.setRequestHeader('Accept', 'application/json');
    var data = obj?JSON.stringify(obj):''
    xhr.send(data)
}
```

包含四个参数。verb，定义了使用HTTP的动作（GET，POST，PUT，DELETE）。第二个参数是作为基础地址的后缀（例如’[http://localhost:5000/colors](http://localhost:5000/colors)'）。第三个参数是可选对象，作为JSON数据发送给服务的数据。最后一个选项是定义当响应返回时的回调。回调接收一个响应数据的响应对象。

在我们发送请求前，我们需要明确我们发送和接收的JSON数据修改的请求头。

```
// colorservice.js
function get_colors(cb) {
    // GET http://localhost:5000/colors
    request('GET', null, null, cb)
}

function create_color(entry, cb) {
    // POST http://localhost:5000/colors
    request('POST', null, entry, cb)
}

function get_color(name, cb) {
    // GET http://localhost:5000/colors/<name>
    request('GET', name, null, cb)
}

function update_color(name, entry, cb) {
    // PUT http://localhost:5000/colors/<name>
    request('PUT', name, entry, cb)
}

function delete_color(name, cb) {
    // DELETE http://localhost:5000/colors/<name>
    request('DELETE', name, null, cb)
}
```

这些代码在服务实现中。在UI中我们使用服务来实现我们的命令。我们有一个存储id的ListModel和存储数据的gridModel为GridView提供数据。命令使用Button元素来发送。

读取服务器颜色链表。

```
// rest.qml
import "colorservice.js" as Service
...
// read colors command
Button {
    text: 'Read Colors';
    onClicked: {
        Service.get_colors( function(resp) {
            print('handle get colors resp: ' + JSON.stringify(resp));
            gridModel.clear();
            var entries = resp.data;
            for(var i=0; i<entries.length; i++) {
                gridModel.append(entries[i]);
            }
        });
    }
}
```

在服务器上创建一个新的颜色。

```
// rest.qml
import "colorservice.js" as Service
...
// create new color command
Button {
    text: 'Create New';
    onClicked: {
        var index = gridModel.count-1
        var entry = {
            name: 'color-' + index,
            value: Qt.hsla(Math.random(), 0.5, 0.5, 1.0).toString()
        }
        Service.create_color(entry, function(resp) {
            print('handle create color resp: ' + JSON.stringify(resp))
            gridModel.append(resp)
        });
    }
}
```

基于名称读取一个颜色。

```
// rest.qml
import "colorservice.js" as Service
...
// read last color command
Button {
    text: 'Read Last Color';
    onClicked: {
        var index = gridModel.count-1
        var name = gridModel.get(index).name
        Service.get_color(name, function(resp) {
            print('handle get color resp:' + JSON.stringify(resp))
            message.text = resp.value
        });
    }
}
```

基于颜色名称更新服务器上的一个颜色。

```
// rest.qml
import "colorservice.js" as Service
...
// update color command
Button {
    text: 'Update Last Color'
    onClicked: {
        var index = gridModel.count-1
        var name = gridModel.get(index).name
        var entry = {
            value: Qt.hsla(Math.random(), 0.5, 0.5, 1.0).toString()
        }
        Service.update_color(name, entry, function(resp) {
            print('handle update color resp: ' + JSON.stringify(resp))
            var index = gridModel.count-1
            gridModel.setProperty(index, 'value', resp.value)
        });
    }
}
```

基于颜色名称删除一个颜色。

```
// rest.qml
import "colorservice.js" as Service
...
// delete color command
Button {
    text: 'Delete Last Color'
    onClicked: {
        var index = gridModel.count-1
        var name = gridModel.get(index).name
        Service.delete_color(name)
        gridModel.remove(index, 1)
    }
}
```

在CRUD（创建，读取，更新，删除）操作使用REST接口。也可以使用其它的方法来创建web服务接口。可以基于模块，每个模块都有自己的后缀。可以使用JSON RPC（[http://www.jsonrpc.org/](http://www.jsonrpc.org/)）来定义接口。当然基于XML的接口也可以使用，但是JSON在作为JavaScript部分解析进QML/JS中更有优势。
