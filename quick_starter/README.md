# Quick Starter
Quick Starter

**注意**

**最后一次构建：2014年1月20日下午18:00。**

**这章的源代码能够在[assetts folder](http://qmlbook.org/assets)找到。**

这章概述了QML语言，Qt5中大量使用了这种声明用户界面的语言。我们将会讨论QML语言，一个树形结构的元素，跟着是一些最基本的元素概述。然后我们会简短的介绍怎样创建我们自己的元素，这些元素被叫做组件，并如何使用属性操作来转换元素。最后我们会介绍如何对元素进行布局，如何向用户提供输入。

4.7 输入元素（Input Element）

我们已经使用过MouseArea（鼠标区域）作为鼠标输入元素。这里我们将更多的介绍关于键盘输入的一些东西。我们开始介绍文本编辑的元素：TextInput（文本输入）和TextEdit（文本编辑）。
4.7.1 文本输入（TextInput）

文本输入允许用户输入一行文本。这个元素支持使用正则表达式验证器来限制输入和输入掩码的模式设置。


用户可以通过点击TextInput来改变焦点。为了支持键盘改变焦点，我们可以使用KeyNavigation（按键向导）这个附加属性。

KeyNavigation（按键向导）附加属性可以预先设置一个元素id绑定切换焦点的按键。
一个文本输入元素（text input element）只显示一个闪烁符和已经输入的文本。用户需要一些可见的修饰来鉴别这是一个输入元素，例如一个简单的矩形框。当你放置一个TextInput（文本输入）在一个元素中时，你需要确保其它的元素能够访问它导出的大多数属性。
我们提取这一段代码作为我们自己的组件，称作TLineEditV1用来重复使用。

注意
如果你想要完整的导出TextInput元素，你可以使用property alias input: input来导出这个元素。第一个input是属性名字，第二个input是元素id。
我们使用TLineEditV1组件重写了我们的KeyNavigation（按键向导）的例子。


尝试使用Tab按键来导航，你会发现焦点无法切换到input2上。这个例子中使用focus:true的方法不正确，这个问题是因为焦点被转移到input2元素时，包含TlineEditV1的顶部元素接收了这个焦点并且没有将焦点转发给TextInput（文本输入）。为了防止这个问题，QML提供了FocusScope（焦点区域）。
4.7.2 焦点区域（FocusScope）

一个焦点区域（focus scope）定义了如果焦点区域接收到焦点，它的最后一个使用focus:true的子元素接收焦点，它将会把焦点传递给最后申请焦点的子元素。我们创建了第二个版本的TLineEdit组件，称作TLineEditV2，使用焦点区域（focus scope）作为根元素。

现在我们的例子将像下面这样：

按下Tab按键可以成功的在两个组件之间切换焦点，并且能够正确的将焦点锁定在组件内部的子元素中。
4.7.3 文本编辑（TextEdit）

文本编辑（TextEdit）元素与文本输入（TextInput）非常类似，它支持多行文本编辑。它不再支持文本输入的限制，但是提供了已绘制文本的大小查询（paintedHeight，paintedWidth）。我们也创建了一个我们自己的组件TTextEdit，可以编辑它的背景，使用focus scope（焦点区域）来更好的切换焦点。

你可以像下面这样使用这个组件：


4.7.4 按键元素（Key Element）

附加属性key允许你基于某个按键的点击来执行代码。例如使用up，down按键来移动一个方块，left，right按键来旋转一个元素，plus，minus按键来缩放一个元素。


4.8 高级用法

后续添加。
