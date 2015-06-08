# 编译系统（Build Systems）

在不同的平台上稳定的编译软件是一个复杂的任务。你将会遇到不同环境下的不同编译器，路径和库变量的问题。Qt的目的是防止应用开发者遭遇这些跨平台问题。为了完成这个任务，Qt引进了```qmake```编译文件生成器。```qmake```操作以```.pro```
结尾的项目文件。这个项目文件包含了关于应用程序的说明和需要读取的资源文件。用qmake执行这个项目文件会为你生成一个在unix和mac的```Makefile```
,如果在windows下使用mingw编译工具链也会生成。否则可能会创建一个visual studio项目或者一个xcode项目。

在unix下使用Qt编译如下：

```
$ edit myproject.pro
$ qmake // generates Makefile
$ make
```

Qt也允许你使用影子编译。影子编译会在你的源码位置外的路径进行编译。假设我们有一个myproject文件夹，里面有一个myproject.pro文件。如下输入命令：

```
$ mkdir build
$ cd build
$ qmake ../myproject/myproject.pro
```

我们创建一个编译文件夹并且在这个编译文件中使用qmake指向我们项目文件夹中的项目文件。这将配置makefile使用编译文件夹替代我们的源代码文件夹来存放所有的编译中间件和结果。这允许我们同时为不同的qt版本和编译配置创建不同的编译文件夹并且不会弄乱我们的源代码文件夹。

当你使用Qt Creator时，它会在后代为你做这些事情，通常你不在需要担心这些步骤。对于比较大的项目，建议使用命令行方式来编译你的Qt项目可以更加深入的了解编译流。

