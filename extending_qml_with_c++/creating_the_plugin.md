# 创建插件（Creating the plugin）

Qt Creator包含了一个创建```QtQuick 2 QML Extension
 Plugin```向导，我们使用它来创建一个叫做```fileio```
的插件，这个插件包含了一个从```org.example.io```中启动的```FileIO```对象。

插件类源于```QQmlExtensionPlugin```，并且实现了```registerTypes()```
函数。```Q_PLUGIN_METADATA```是强制标识这个插件作为一个qml扩展插件。除此之外没有其它特殊的地方了。

```
#ifndef FILEIO_PLUGIN_H
#define FILEIO_PLUGIN_H

#include <QQmlExtensionPlugin>

class FileioPlugin : public QQmlExtensionPlugin
{
    Q_OBJECT
    Q_PLUGIN_METADATA(IID "org.qt-project.Qt.QQmlExtensionInterface")

public:
    void registerTypes(const char *uri);
};

#endif // FILEIO_PLUGIN_H
```

在实现```registerTypes```中我们使用```qmlRegisterType```函数注册了我们的FileIO类。

```
#include "fileio_plugin.h"
#include "fileio.h"

#include <qqml.h>

void FileioPlugin::registerTypes(const char *uri)
{
    // @uri org.example.io
    qmlRegisterType<FileIO>(uri, 1, 0, "FileIO");
}
```

有趣的是我们不能在这里看到模块统一资源标识符（例如```org.example.io```）。这似乎是从外面设置的。

看你查找你的项目文件夹是，你会发现一个qmldir文件。这个文件指定了你的qml插件内容或者最好是你插件中关于QML的部分。它看起来应该像这样。

```
module org.example.io
plugin fileio
```

模块是统一资源标识符，在统一标识符下插件能够被其它插件获取，并且插件行必须与插件文件名完全相同（在mac下，它将是```libfileio_debug.dylib```存在于文件系统上，```fileio```在```qmldir```中）。这些文件由Qt Creator基于给定的信息创建。模块的标识符在```.pro```文件中同样可用。用来构建安装文件夹。

当你在构建文件夹中调用```make install```时，将会拷贝库文件到Qt```qml```
文件夹中（在Qt5.4之后mac上这将在```~/Qt/5.4/clang_64/qml```文件夹中。这个路径依赖Qt按住那个位置，并且使用系统上的编译器）。你将会在```org/example/io```文件夹中发现库文件。目前包含两个文件。

```
libfileio_debug.dylib
qmldir
```

当导入一个叫做```org.example.io```的模块时，qml引擎将会在导入路径下查找一个可用模块并且尝试使用qmldir文件定位```org/example/io```路径。qmldir会告诉引擎使用哪个模块标识符加在哪个库作为qml扩展插件。两个模块使用相同的统一标识符将会相互覆盖。
