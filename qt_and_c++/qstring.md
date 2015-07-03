# QString

通常在Qt中文本操作是基于unicode完成的。你需要使用```QString```类来完成这个事情。它包含了很多好用的功能函数，这与其它流行的框架类似。对于8位的数据你通常需要使用```QByteArray```类，对于ASCII校验最好使用```QLatin1String```来暂存。对于一个字符串链你可以使用```QList<QString>```或者```QStringList```类（派生自```QList<QString>```）。

这里有一些例子介绍了如何使用```QString```类。QString可以在栈上创建，但是它的数据存储在堆上。分配一个字符串数据到另一个上，不会产生拷贝操作，只是创建了数据的引用。这个操作非常廉价让开发者更专注于代码而不是内存操作。```QString```使用引用计数的方式来确定何时可以安全的删除数据。这个功能叫做[隐式共享](http://doc.qt.io/qt-5//implicit-sharing.html)，在Qt的很多类中都用到了它。

```
    QString data("A,B,C,D"); // create a simple string
    // split it into parts
    QStringList list = data.split(",");
    // create a new string out of the parts
    QString out = list.join(",");
    // verify both are the same
    QVERIFY(data == out);
    // change the first character to upper case
    QVERIFY(QString("A") == out[0].toUpper());
```

这里我们将展示如何将一个字符串转换为数字，将一个数字转换为字符串。也有一些方便的函数用于float或者double和其它类型的转换。只需要在Qt帮助文档中就可以找到这些使用方法。

```
    // create some variables
    int v = 10;
    int base = 10;
    // convert an int to a string
    QString a = QString::number(v, base);
    // and back using and sets ok to true on success
    bool ok(false);
    int v2 = a.toInt(&ok, base);
    // verify our results
    QVERIFY(ok == true);
    QVERIFY(v = v2);
```

通常你需要参数化文本。例如使用```QString("Hello" + name)```
,一个更加灵活的方法是使用```arg```标记目标，这样即使在翻译时也可以保证目标的变化。

```
    // create a name
    QString name("Joe");
    // get the day of the week as string
    QString weekday = QDate::currentDate().toString("dddd");
    // format a text using paramters (%1, %2)
    QString hello = QString("Hello %1. Today is %2.").arg(name).arg(weekday);
    // This worked on Monday. Promise!
    if(Qt::Monday == QDate::currentDate().dayOfWeek()) {
        QCOMPARE(QString("Hello Joe. Today is Monday."), hello);
    } else {
        QVERIFY(QString("Hello Joe. Today is Monday.") !=  hello);
    }
```

有时你需要在你的代码中直接使用unicode字符。你需要记住如何在使用```QChar```和```QString```类来标记它们。

```
    // Create a unicode character using the unicode for smile :-)
    QChar smile(0x263A);
    // you should see a :-) on you console
    qDebug() << smile;
    // Use a unicode in a string
    QChar smile2 = QString("\u263A").at(0);
    QVERIFY(smile == smile2);
    // Create 12 smiles in a vector
    QVector<QChar> smilies(12);
    smilies.fill(smile);
    // Can you see the smiles
    qDebug() << smilies;
```

上面这些示例展示了在Qt中如何轻松的处理unicode文本。对于非unicode文本，QByteArray类同样有很多方便的函数可以使用。阅读Qt帮助文档中QString部分，它有一些很好的示例。
