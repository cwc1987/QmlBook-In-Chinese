# QObject对象（The QObject）

正如介绍中描述的，```QObject```是Qt的内省机制。在Qt中它几乎是所有类的基类。值类型除外，例如```QColor```，```QString```和```QList```。

Qt对象是一个标准的C++对象，但是它具有更多的功能。可以从两个方向来深入探讨：内省和内存管理。内省意味着Qt对象知道它的类名，它与其它类的关系，以及它的方法和属性。内存管理意味着每个Qt对象都可以成为是其它子对象的父对象。父对象拥有子对象，当父对象销毁时，它也会负责销毁它的子对象。

理解```QObject```的能力如何影响一个类最好的方法是使用Qt的类来替换一个典型的C++类。如下所示的代表一个普通的类。

类```Person```是一个数据类，包含了一个名字和性别属性。```Person```使用Qt的对象系统来添加一个元信息到c++类中。它允许使用```Person```对象的用户连接槽函数并且当属性变化时获得通知。

```
class Person : public QObject
{
    Q_OBJECT // enabled meta object abilities

    // property declarations required for QML
    Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)
    Q_PROPERTY(Gender gender READ gender WRITE setGender NOTIFY genderChanged)

    // enables enum introspections
    Q_ENUMS(Gender)

public:
    // standard Qt constructor with parent for memory management
    Person(QObject *parent = 0);

    enum Gender { Unknown, Male, Female, Other };

    QString name() const;
    Gender gender() const;

public slots: // slots can be connected to signals
    void setName(const QString &);
    void setGender(Gender);

signals: // signals can be emitted
    void nameChanged(const QString &name);
    void genderChanged(Gender gender);

private:
    // data members
    QString m_name;
    Gender m_gender;
};
```

构造函数传入父对象到超类中并且初始化成员变量。Qt的值类型类会自动初始化。在这个例子中```QString```
将会初始化为一个空字符串（```QString::isNull()```）并且性别成员变量会明确的初始化为未知性别。

```
Person::Person(QObject *parent)
    : QObject(parent)
    , m_gender(Person::Unknown)
{
}
```

获取函数在命名在属性后并且是一个简单的```const```函数。使用设置属性函数当属性被改变时会发送改变信号。为此我们插入一个保护用来比较当前值与新值。只有在值不同时我们指定给成员变量的值才会生效，并且发送改变信号。

```
QString Person::name() const
{
    return m_name;
}

void Person::setName(const QString &name)
{
    if (m_name != name) // guard
    {
        m_name = name;
        emit nameChanged(m_name);
    }
}
```

类通过继承```QObject```，我们获得了元对象能力，我们可以尝试使用```metaObject()```的方法。例如从对象中检索类名。

```
Person* person = new Person();
person->metaObject()->className(); // "Person"
Person::staticMetaObject.className(); // "Person"
```

QObject基类和元对象还有其它很多功能。详情请查看```QMetaObject```文档获取更多信息。




