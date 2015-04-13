# JavaScript语法（The Language）

这章不会对JavaScript作详细的介绍。有其它的书可以参考，请访问[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)

JavaScript表面上是一种非常通用的语言，与许多其它语言没有什么不同：

```
function countDown() {
  for(var i=0; i<10; i++) {
    console.log('index: ' + i)
  }
}

function countDown2() {
  var i=10;
  while( i>0 ) {
    i--;
  }
}
```

但是注意JS有函数作用域，没有C++中的块作用域（查看[Functions and function scope](https://developer.mozilla.org/it/docs/Web/JavaScript/Reference/Functions_and_function_scope)）。

语句if ... else，break，continue也可以使用。switch相对于C++中只可以切换整数值，JavaScript可以切换其它类型的值：

```
function getAge(name) {
  // switch over a string
  switch(name) {
  case "father":
    return 58;
  case "mother":
    return 56;
  }
  return unknown;
}
```

JS可以将几种值认为是false，如false，0，“”，undefined，null。例如一个函数范围默认值undefined。使用‘===’操作符验证是否为false。‘==’等于操作将会对类型转换做验证。如果条件允许，直接使用等于操作符‘===’可以更快更好的验证一致性（查看[Comparison operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)）。

在JavaScript底层有它自己的实现方式，例如数组：

```
function doIt() {
  var a = [] // empty arrays
  a.push(10) // addend number on arrays
  a.push("Monkey") // append string on arrays
  console.log(a.length) // prints 2
  a[0] // returns 10
  a[1] // returns Monkey
  a[2] // returns undefined
  a[99] = "String" // a valid assignment
  console.log(a.length) // prints 100
  a[98] // contains the value undefined
}
```

当然对比C++或者Java这种OO语言，JS的工作方式是不同的。JS不是一种纯粹的OO语言，它也可以称作基于原型语言。每个对象都有一个原型对象。对象是基于这个原型对象创建的。请阅读这本关于JavaScript的书了解更多[Javascript the Good Parts by Douglas Crockford ](http://javascript.crockford.com/)。

可以使用在线JS控制台或者小片断的QML代码来测试小的JS片段代码：

```
import QtQuick 2.0

Item {
  function runJS() {
    console.log("Your JS code goes here");
  }
  Component.onCompleted: {
    runJS();
  }
}
```






