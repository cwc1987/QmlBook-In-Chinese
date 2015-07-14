# 创建插件（Creating the plugin）

Qt Creator包含了一个创建QtQuick 2 QML Extension Plugin向导，我们使用它来创建一个叫做fileio的插件，这个插件包含了一个从org.example.io中启动的FileIO对象。

插件类源于QQmlExtensionPlugin，并且实现了registerTypes()函数。Q_PLUGIN_METADATA是强制标识这个插件作为一个qml扩展插件。除此之外没有其它特殊的地方了。

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

在实现registerTypes中我们使用qmlRegisterType函数注册了我们的FileIO类。

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

有趣的是我们不能再这里看到模块标识符（例如org.example.io）。这似乎是从外面设置的。

看你查找你的项目文件夹是，你会发现一个qmldir文件。这个文件指定了你的qml插件内容或者最好是你插件中关于QML的部分。它看起来应该像这样。

```
module org.example.io
plugin fileio
```


The module is the URI under which your plugin is reachable by others and the plugin line must be identical with your plugin file name (under mac this would be libfileio_debug.dylib on the file system and fileio in the qmldir). These files where created by Qt Creator based on the given information. The module uri is also available in the .pro file. There is is used to build up the install directory.

When you call make install in your build folder the library will be copied into the Qt qml folder (for Qt 5.4 on mac this would be “~/Qt/5.4/clang_64/qml”. The exact path depends on you Qt installation location and the used compiler on your system). There you will find a the library inside the “org/example/io” folder. The content are these two files currently


