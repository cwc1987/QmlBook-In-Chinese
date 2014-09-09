# HTTP请求（HTTP Requests）

从c++方面来看，Qt中完成http请求通常是使用QNetworkRequest和QNetworkReply，然后使用Qt/C++将响应推送到集成的QML。所以我们尝试使用QtQuick的工具给我们的网络信息尾部封装了小段信息，然后推送这些信息。为此我们使用一个帮助对象来构造http请求，和循环响应。它使用java脚本的XMLHttpRequest对象的格式。

XMLHttpRequest对象允许用户注册一个响应操作函数和一个链接。一个请求能够使用http动作来发送（如get，post，put，delete，等等）。当响应到达时，会调用注册的操作函数。操作函数会被调用多次。每次调用请求的状态都已经改变（例如信息头部已接收，或者响应完成）。

下面是一个简短的例子：

```
function request() {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (xhr.readyState === XMLHttpRequest.HEADERS_RECEIVED) {
            print('HEADERS_RECEIVED');
        } else if(xhr.readyState === XMLHttpRequest.DONE) {
            print('DONE');
        }
    }
    xhr.open("GET", "http://example.com");
    xhr.send();
}
```

从一个响应中你可以获取XML格式的数据或者是原始文本。可以遍历XML结果但是通常使用原始文本来匹配JSON格式响应。使用JSON.parse(text）可以JSON文档将转换为JS对象使用。

```
...
} else if(xhr.readyState === XMLHttpRequest.DONE) {
    var object = JSON.parse(xhr.responseText.toString());
    print(JSON.stringify(object, null, 2));
}
```

在响应操作中，我们访问原始响应文本并且将它转换为一个javascript对象。JSON对象是一个可以使用的JS对象（在javascript中，一个对象可以是对象或者一个数组）。

**注意**

**toString()转换似乎让代码更加稳定。在不使用显式的转换下我有几次都解析错误。不确定是什么问题引起的。**

## 11.3.1 Flickr调用（Flickr Call）

让我们看看更加真实的例子。一个典型的例子是使用网络相册服务来取得公共订阅中新上传的图片。我们可以使用[http://api.flicker.com/services/feeds/photos_public.gne](http://api.flicker.com/services/feeds/photos_public.gne)链接。不幸的是它默认返回XML流格式的数据，在qml中可以很方便的使用XmlListModel来解析。为了达到只关注JSON数据的目的，我们需要在请求中附加一些参数可以得到JSON响应：[http://api.flickr.com/services/feeds/photo_public.gne?format=json&nojsoncallback=1](http://api.flickr.com/services/feeds/photo_public.gne?format=json&nojsoncallback=1)。这将会返回一个没有JSON回调的JSON响应。

**注意**
**一个JSON回调将JSON响应包装在一个函数调用中。这是一个HTML编程中的快捷方式，使用脚本标记来创建一个JSON请求。响应将触发本地定义的回调函数。在QML中没有JSON回调的工作机制。**

使用curl来查看响应：

```
curl "http://api.flickr.com/services/feeds/photos_public.gne?format=json&nojsoncallback=1&tags=munich"
```

响应如下：

```
{
    "title": "Recent Uploads tagged munich",
    ...
    "items": [
        {
        "title": "Candle lit dinner in Munich",
        "media": {"m":"http://farm8.staticflickr.com/7313/11444882743_2f5f87169f_m.jpg"},
        ...
        },{
        "title": "Munich after sunset: a train full of \"must haves\" =",
        "media": {"m":"http://farm8.staticflickr.com/7394/11443414206_a462c80e83_m.jpg"},
        ...
        }
    ]
    ...
}
```

JSON文档已经定义了结构体。一个对象包含一个标题和子项的属性。标题是一个字符串，子项是一组对象。当转换文本为一个JSON文档后，你可以单独访问这些条目，它们都是可用的JS对象或者结构体数组。

```
// JS code
obj = JSON.parse(response);
print(obj.title) // => "Recent Uploads tagged munich"
for(var i=0; i<obj.items.length; i++) {
    // iterate of the items array entries
    print(obj.items[i].title) // title of picture
    print(obj.items[i].media.m) // url of thumbnail
}
```

我们可以使用obj.items数组将JS数组作为链表视图的模型，试着完成这个操作。首先我们需要取得响应并且将它转换为可用的JS对象。然后设置response.items属性作为链表视图的模型。

```
function request() {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if(...) {
            ...
        } else if(xhr.readyState === XMLHttpRequest.DONE) {
            var response = JSON.parse(xhr.responseText.toString());
            // set JS object as model for listview
            view.model = response.items;
        }
    }
    xhr.open("GET", "http://api.flickr.com/services/feeds/photos_public.gne?format=json&nojsoncallback=1&tags=munich");
    xhr.send();
}
```

下面是完整的源代码，当组件加载完成后，我们创建请求。然后使用请求的响应作为我们链表视图的模型。

```
import QtQuick 2.0

Rectangle {
    width: 320
    height: 480
    ListView {
        id: view
        anchors.fill: parent
        delegate: Thumbnail {
            width: view.width
            text: modelData.title
            iconSource: modelData.media.m
        }
    }

    function request() {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.HEADERS_RECEIVED) {
                print('HEADERS_RECEIVED')
            } else if(xhr.readyState === XMLHttpRequest.DONE) {
                print('DONE')
                var json = JSON.parse(xhr.responseText.toString())
                view.model = json.items
            }
        }
        xhr.open("GET", "http://api.flickr.com/services/feeds/photos_public.gne?format=json&nojsoncallback=1&tags=munich");
        xhr.send();
    }

    Component.onCompleted: {
        request()
    }
}
```

当文档完整加载后（Component.onCompleted）,我们从Flickr请求最新的订阅内容。我们解析JSON的响应并且设置item数组作为我们视图的模型。链表视图有一个代理可以在一行中显示图标缩略图和标题文本。

另一种方法是添加一个ListModel，并且将每个子项添加到链表模型中。为了支持更大的模型，需要支持分页和懒加载。
