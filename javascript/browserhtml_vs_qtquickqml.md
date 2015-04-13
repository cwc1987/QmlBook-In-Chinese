# 浏览器/HTML与QtQuick/QML对比（Browser/HTML vs QtQuick/QML）

浏览器在运行时渲染HTML，执行HTML中相关的JavaScript。现今的web应用中相对于HTML包含了更多的JavaScript。浏览器中JavaScript运行在一些浏览器附加的标准ECMAScript环境。一个典型的浏览器中的JS环境知道访问浏览器窗口的窗口对象。也简单的基于JQuery的DOM选择器来提供CSS选择器。额外使用setTimeout函数在超时时调用函数。除了这些，JS存在于一个标准的JavaScript环境，类似于QML/JS。

不同的是JS出现在HTML与QML中的方式。在HTML中，你只能在事件操作（event handlers），例如页面加载（page loaded），鼠标点击（mouse pressed）中添加JS。例如通常在页面加载中初始化你的JS，这在QML中与组件加载完成（Component.onCompleted）类似。例如你不能使用JS来绑定属性（至少不是直接绑定，AngularJS增强了DOM树允许这种操作，但这和典型HTML相去甚远）。

所以在QML中JS是一种更加优秀的语言，并且与QML的渲染树高度集成。使得语言更具有可读性。除了这些，开发过HTML/JS应用程序的人会觉得在QML/JS中开发非常容易上手。

