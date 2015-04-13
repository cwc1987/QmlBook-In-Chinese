# JavaScript

JavaScript是web客户端开发的通用语言。基于node js它也开始引导web服务器的开发。因此它使非常适合在声明式QML语言上添加的命令性语言。QML本身作为一个申明式语言用于表达用户界面层次，但是这仅限于表达操作。有时你需要一个方法表达业务，使用JavaScript来完成。

**注意**

**在Qt社区有一个开放性的问题是在目前Qt程序中关于混合使用QML/JS/QtC++的正确性。通常建议的混合方式是在你的应用程序中将JS部分限制在最小，将你的业务逻辑部分放在QtC++中，UI逻辑放在QML/JS中。**

**这本书趋向这种边界的划分，通常对于一个产品的开发这不一定是正确的混合方式，不是对于所有人都适用。最重要的是根据你的团队技能和个人品味而定。在接受推荐的时候保持你的怀疑。**

下面有一个简短的例子是关于如何在QML中混合适用JS：

```
Button {
  width: 200
  height: 300
  property bool toggle: false
  text: "Click twice to exit"

  // JS function
  function doToggle() {
    toggle = !toggle
  }

  onTriggered: {
    // this is JavaScript
    doToggle();
    if(toggle) {
      Qt.quit()
    }
  }
}
```

因此在QML中JavaScript作为一个单独的JS函数，作为一个JS模块可以在很多地方使用，它可以与每一个右边的属性绑定。

```
import "util.js" as Util // import a pure JS module

Button {
  width: 200
  height: width*2 // JS on the right side of property binding

  // standalone function (not really useful)
  function log(msg) {
    console.log("Button> " + msg);
  }

  onTriggered: {
    // this is JavaScript
    log();
    Qt.quit();
  }
}
```

在使用QML定义用户界面时，使用JavaScript完成功能。那么你需要写多少的JavaScript呢？这取决于你的风格和你对JS开发的熟悉程度。JS是一种松散型语言，这使得你很难发现类型缺陷。函数参数接受不同类型的变量值，会导致非常难发现严重的Bug。发现缺陷的方法是严格的单元测试或者验收测试。因此如果你在JS中开发真正的逻辑（不是粘贴代码）你应该使用测试优先的方法。通常使用这种混合开发非常成功的团队（Qt/C++与QML/JS），他们都会最小化前段逻辑中使用的JS，在后端Qt C++中完成更加复杂的工作。后端遵循严格的单元测试，这样前段的开发者可以信任这些代码并且专注于用户界面的需求。

**注意**

**通常：后端开发者由功能驱动，前段开发者由用户场景驱动。**





