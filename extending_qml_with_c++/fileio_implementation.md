# FileIO实现（FileIO Implementation）

类```FileIO```实现很简单。记住编程接口我们想要创建的像这样。

```
class FileIO : public QObject {
    ...
    Q_PROPERTY(QUrl source READ source WRITE setSource NOTIFY sourceChanged)
    Q_PROPERTY(QString text READ text WRITE setText NOTIFY textChanged)
    ...
public:
    Q_INVOKABLE void read();
    Q_INVOKABLE void write();
    ...
}
```

我们将保留属性，因为它们是简单的设置者和获取者。

读取方法在读取模式下打开一个文件并且使用一个文本流读取数据。

```
void FileIO::read()
{
    if(m_source.isEmpty()) {
        return;
    }
    QFile file(m_source.toLocalFile());
    if(!file.exists()) {
        qWarning() << "Does not exits: " << m_source.toLocalFile();
        return;
    }
    if(file.open(QIODevice::ReadOnly)) {
        QTextStream stream(&file);
        m_text = stream.readAll();
        emit textChanged(m_text);
    }
}
```

当文本变化时，使用```emit textChanged(m_text)```需要通知其它对象这个变化。否则属性绑定无法工作。

写入方法相同，但是在写入模式下打开文件，使用文本流写入内容。

```
void FileIO::write()
{
    if(m_source.isEmpty()) {
        return;
    }
    QFile file(m_source.toLocalFile());
    if(file.open(QIODevice::WriteOnly)) {
        QTextStream stream(&file);
        stream << m_text;
    }
}
```

最后不要忘记调用make install。否则你的插件文件不会拷贝到qml文件夹，qml引擎无法定位模块。

由于读取和写入会阻塞程序运行，你只能使用FileIO处理小型文本，否则会阻塞Qt的UI线程运行。这里一定要注意！
