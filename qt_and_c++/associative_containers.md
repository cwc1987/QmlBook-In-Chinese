# 组合容器（Associative Containers）

映射，字典或者集合是组合容器的例子。它们使用一个键来保存一个值。它们可以快速的查询它们的元素。我们将展示使用最多的组合容器```QHash```，同时也会展示一些C++11新的特性。

```
    QHash<QString, int> hash({{"b",2},{"c",3},{"a",1}});
    qDebug() << hash.keys(); // a,b,c - unordered
    qDebug() << hash.values(); // 1,2,3 - unordered but same as order as keys

    QVERIFY(hash["a"] == 1);
    QVERIFY(hash.value("a") == 1);
    QVERIFY(hash.contains("c") == true);

    { // JAVA iterator
        int sum =0;
        QHashIterator<QString, int> i(hash);
        while (i.hasNext()) {
            i.next();
            sum+= i.value();
            qDebug() << i.key() << " = " << i.value();
        }
        QVERIFY(sum == 6);
    }

    { // STL iterator
        int sum = 0;
        QHash<QString, int>::const_iterator i = hash.constBegin();
        while (i != hash.constEnd()) {
            sum += i.value();
            qDebug() << i.key() << " = " << i.value();
            i++;
        }
        QVERIFY(sum == 6);
    }

    hash.insert("d", 4);
    QVERIFY(hash.contains("d") == true);
    hash.remove("d");
    QVERIFY(hash.contains("d") == false);

    { // hash find not successfull
        QHash<QString, int>::const_iterator i = hash.find("e");
        QVERIFY(i == hash.end());
    }

    { // hash find successfull
        QHash<QString, int>::const_iterator i = hash.find("c");
        while (i != hash.end()) {
            qDebug() << i.value() << " = " << i.key();
            i++;
        }
    }

    // QMap
    QMap<QString, int> map({{"b",2},{"c",2},{"a",1}});
    qDebug() << map.keys(); // a,b,c - ordered ascending

    QVERIFY(map["a"] == 1);
    QVERIFY(map.value("a") == 1);
    QVERIFY(map.contains("c") == true);

    // JAVA and STL iterator work same as QHash
```

