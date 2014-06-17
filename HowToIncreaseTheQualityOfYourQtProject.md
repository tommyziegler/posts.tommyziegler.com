# How to increase the quality of your Qt Project!

In this article I wanna describe how to start a new successful Qt Project. The goal should be to start C++ projects nowadays similar like in some other fields (Test-Driven, with Continious Integration, Code Quality Metrics etc.)

...

## Test-Driven Development

The most important part is Test-Driven-Development, the team should learn how to create code and features ONLY with the needed tests. I know that's really hard at the beginning. The easiest way for me at the beginning was to write the test code after I wrote the functionality, also if the Test-first approach should be your target.

There exists a useful Tutorial on the Qt-Project site: http://qt-project.org/doc/qt-4.8/qtestlib-tutorial1.html


### Writing tests
Additional this useful selfwritten Qt-Cpp macro helps a lot to easy write quickly new test classes. 
```C++
#ifndef TOMMYZIEGLERSTESTFRAMEWORKQT_H
#define TOMMYZIEGLERSTESTFRAMEWORKQT_H

#include <QList>
#include <QString>
#include <QSharedPointer>

#include <QDir>
#include <QtTest/QtTest>

namespace TommyZieglersTestFrameworkQt
{
    typedef QList<QObject*> TestList;
    
    inline TestList& testList()
    {
        static TestList list;
        return list;
    }

    inline bool findObject(QObject* object)
    {
        TestList& list = testList();
        if (list.contains(object))
        {
            return true;
        }
        
        foreach (QObject* test, list)
        {
            if (test->objectName() == object->objectName())
            {
                return true;
            }
        }
        return false;
    }

    inline void addTest(QObject* object)
    {
        TestList& list = testList();

        if (!findObject(object))
        {
            list.append(object);
        }
    }

    inline int run(int argc, char *argv[])
    {
        int result(0);

        if(argc > 1) {
            if( (QString::compare(QString(argv[1]), "-xml", Qt::CaseInsensitive) == 0) ||
                (QString::compare(QString(argv[1]), "-xunitxml", Qt::CaseInsensitive) == 0) ){

                foreach (QObject* test, testList())
                {
                    QString testClassName = test->metaObject()->className();

                    QString nameFile(testClassName);
                    nameFile += ".xml";

                    QString filePath("TommyZieglersTestFrameworkQt");
                    if(!QDir(filePath).exists())
                        QDir().mkdir(filePath);

                    QString outFilePath;
                    outFilePath += filePath;
                    outFilePath += "/";
                    outFilePath += nameFile;

                    QStringList arguments;
                    arguments << " "
                              << QString(argv[1])
                              << "-o"
                              << outFilePath;


                    result |= QTest::qExec(test, arguments);
                }

                return result;
            }
        }

        foreach (QObject* test, testList())
        {
            QStringList arguments;
            arguments << " ";
            result |= QTest::qExec(test, arguments);
        }
        return result;
    }
}

template <class T>
class Test
{
public:
    QSharedPointer<T> child;

    Test(const QString& name) : child(new T)
    {
        child->setObjectName(name);
        TommyZieglersTestFrameworkQt::addTest(child.data());
    }
};

#define DECLARE_TEST(className) static Test<className> t(#className);

#endif // TOMMYZIEGLERSTESTFRAMEWORKQT_H
```

The main method can look then like this: 
```C++
#include "TommyZieglersTestFrameworkQt.h"

int main(int argc, char *argv[])
{
    return TommyZieglersTestFrameworkQt::run(argc, argv);
}
```

Like this we can write then a test case
```C++
#include "QtTDDTest.h"

QtTDDTest::QtTDDTest(QObject *parent) :
    QObject(parent)
{
}

void QtTDDTest::testSucceed()
{
    QVERIFY2(true == true, "check that true is true");
}

void QtTDDTest::testFail()
{
    QVERIFY2(true == false, "check that true is false");
}
```
```C++
#ifndef QTTDDTEST_H
#define QTTDDTEST_H

#include "TommyZieglersTestFrameworkQt.h"
#include <QObject>

class QtTDDTest : public QObject
{
    Q_OBJECT
public:
    explicit QtTDDTest(QObject *parent = 0);
    
signals:
    
private Q_SLOTS:
    void testSucceed();
    void testFail();
    
};

// Add to automatic test suite
DECLARE_TEST(QtTDDTest)

#endif // QTTDDTEST_H
```
The DECLARE_TEST adds the current test class into the test suite. It's needed to add this line in every new test class which should be executed.

### Running tests

The test program can be runned while development without any parameters. Our output on the Console will look like this:

```
********* Start testing of QtTDDTest *********
Config: Using QTest library 4.8.5, Qt 4.8.5
PASS   : QtTDDTest::initTestCase()
PASS   : QtTDDTest::testSucceed()
FAIL!  : QtTDDTest::testFail() 'true == false' returned FALSE. (check that true is false)
PASS   : QtTDDTest::cleanupTestCase()
Totals: 3 passed, 1 failed, 0 skipped
********* Finished testing of QtTDDTest *********
```
Additional we can start the program also with parameter `-xml` for a normal qt xml result file or with `-xunitxml` to retrieve a xunit xml file. 

**IMPORTANT:** When you wanna read this file with the jenkins xunit test plugin, use the normal `-xml` parameter, because when I tried it the formats wasn't compatible.

## Code Quality Metrics
### Static Code Analyse
The easy way for the static code analyse is the [Cppchecker](http://cppcheck.sourceforge.net) in combination with the [QtProjectTool](http://sourceforge.net/projects/qtprojecttool/).

```
> qpt --tool=cppcheck sample.pro
```

Possible 
```
[qpt running]: /usr/bin/cppcheck —platform=unix32 -I . -f -q —template ‘{file}:{line}:{message}’ —inline-suppr 
                                 —enable=style —enable=unusedFunction —inconclusive /path/one.cpp 
                                                                                    /path/two.cpp
                                                                                    /path/three.cpp

‘/path/one.cpp:3:Unused variable: ret’
‘/path/one.cpp:4:Variable ‘x’ is not assigned a value.’
‘/path/two.cpp:3:Unused variable: ret’
‘/path/two.cpp:4:Variable ‘x’ is not assigned a value.’
‘/path/one.cpp:1:The function ‘x’ is never used.’
’/path/three.cpp:5:Variable ‘x’ is not assigned a value.’
```

## Additional informations

This sample was generated and tested with Qt 4.8.x on Windows and Mac OS X.
