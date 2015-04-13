# JS对象（JS Objects）

在使用JS工作时，有一些对象和方法会被频繁的使用。下面是它们的一个小的集合。

* Math.floor(v),Math.ceil(v),Math.round(v) - 从浮点数获取最大，最小和随机整数
* Math.random() - 创建一个在0到1之间的随机数
* Object.keys(o) - 获取对象的索引值（包括QObject）
* JSON.parse(s), JSON.stringify(o) - 转换在JS对象和JSON字符串
* Number.toFixed(p) - 修正浮点数精度
* Date - 日期时间操作

你可以可以在这里找到它们的使用方法：JavaScript reference

You can find them also at: [JavaScript reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)

下面有一些简单的例子演示了如何在QML中使用JS。会给你一些如何在QML中使用JS一些启发。

**打印所有项的键（Print all keys from QML Item）**

```
Item {
  id: root
  Component.onCompleted: {
    var keys = Object.keys(root);
    for(var i=0; i<keys.length; i++) {
      var key = keys[i];
      // prints all properties, signals, functions from object
      console.log(key + ' : ' + root[key]);
    }
  }
}
```

**转换一个对象为JSON字符串并反转转换（Parse an object to a JSON string and back**）

```
Item {
  property var obj: {
    key: 'value'
  }

  Component.onCompleted: {
    var data = JSON.stringify(obj);
    console.log(data);
    var obj = JSON.parse(data);
    console.log(obj.key); // > 'value'
  }
}
```

**当前时间（Current Date）**

```
Item {
  Timer {
    id: timeUpdater
    interval: 100
    running: true
    repeat: true
    onTriggered: {
      var d = new Date();
      console.log(d.getSeconds());
    }
  }
}
```

**使用名称调用函数（Call a function by name）**

```
Item {
  id: root

  function doIt() {
    console.log("doIt()")
  }

  Component.onCompleted: {
    // Call using function execution
    root["doIt"]();
    var fn = root["doIt"];
    // Call using JS call method (could pass in a custom this object and arguments)
    fn.call()
  }
}
```




