# 顺序容器（Sequential Containers）

链表，队列，数组都是顺序容器。最常用的顺序容器是QList类。它是一个模板类，需要一个类型才能被初始化。它也是隐式共享的，数据存放在堆中。所有的容器类应该被创建在栈上。正常情况下你不需要使用new QList<T>()这样的语句，千万不要使用new来初始化一个容器。

类QList与类QString一样强大，提供了方便的接口来查询数据。下面一个简单的示例展示了如何使用和遍历链表，这里面也使用到了一些C++11的新特性。

```
    // Create a simple list of ints using the new C++11 initialization
    // for this you need to add "CONFIG += c++11" to your pro file.
    QList<int> list{1,2};

    // append another int
    list << 3;

    // We are using scopes to avoid variable name clashes

    { // iterate through list using Qt for each
        int sum(0);
        foreach (int v, list) {
            sum += v;
        }
        QVERIFY(sum == 6);
    }
    { // iterate through list using C++ 11 range based loop
        int sum = 0;
        for(int v : list) {
            sum+= v;
        }
        QVERIFY(sum == 6);
    }

    { // iterate through list using JAVA style iterators
        int sum = 0;
        QListIterator<int> i(list);

        while (i.hasNext()) {
            sum += i.next();
        }
        QVERIFY(sum == 6);
    }

    { // iterate through list using STL style iterator
        int sum = 0;
        QList<int>::iterator i;
        for (i = list.begin(); i != list.end(); ++i) {
            sum += *i;
        }
        QVERIFY(sum == 6);
    }


    // using std::sort with mutable iterator using C++11
    // list will be sorted in descending order
    std::sort(list.begin(), list.end(), [](int a, int b) { return a > b; });
    QVERIFY(list == QList<int>({3,2,1}));


    int value = 3;
    { // using std::find with const iterator
        QList<int>::const_iterator result = std::find(list.constBegin(), list.constEnd(), value);
        QVERIFY(*result == value);
    }

    { // using std::find using C++ lambda and C++ 11 auto variable
        auto result = std::find_if(list.constBegin(), list.constBegin(), [value](int v) { return v == value; });
        QVERIFY(*result == value);
    }
```

