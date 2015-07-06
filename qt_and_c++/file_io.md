# 文件IO（File IO）

通常我们都需要从读写文件。```QFile```是一个```QObject```对象，但是大多数情况下它被创建在栈上。```QFile```包含了通知用户数据可读取信号。它可以异步读取大段的数据，直到整个文件读取完成。为了方便它允许使用阻塞的方式读取数据。这种方法通常用于读取小段数据或者小型文件。幸运的是我们在这些例子中都只使用了小型数据。

除了读取文件内容到内存中可以使用```QByteArray```，你也可以根据读取数据类型使用```QDataStream```或者使用```QTextStream```读取unicode字符串。我们现在来看看如何使用。

```
    QStringList data({"a", "b", "c"});
    { // write binary files
        QFile file("out.bin");
        if(file.open(QIODevice::WriteOnly)) {
            QDataStream stream(&file);
            stream << data;
        }
    }
    { // read binary file
        QFile file("out.bin");
        if(file.open(QIODevice::ReadOnly)) {
            QDataStream stream(&file);
            QStringList data2;
            stream >> data2;
            QCOMPARE(data, data2);
        }
    }
    { // write text file
        QFile file("out.txt");
        if(file.open(QIODevice::WriteOnly)) {
            QTextStream stream(&file);
            QString sdata = data.join(",");
            stream << sdata;
        }
    }
    { // read text file
        QFile file("out.txt");
        if(file.open(QIODevice::ReadOnly)) {
            QTextStream stream(&file);
            QStringList data2;
            QString sdata;
            stream >> sdata;
            data2 = sdata.split(",");
            QCOMPARE(data, data2);
        }
    }
```
