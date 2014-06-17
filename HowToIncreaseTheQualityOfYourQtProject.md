# How to increase the quality of your Qt Project!


In this article I wanna describe how to start a new successful Qt Project. The goal should be to start C++ projects nowadays similar like in some other fields (Test-Driven, with Continious Integration, Code Quality Metrics etc.)

The most important part is Test-Driven-Development, the team should learn how to create code and features ONLY with the needed tests. I know that's really hard at the beginning. The easiest way for me at the beginning was to write the test code after I wrote the functionality, also if the Test-first approach should be your target.


## Test-Driven Development

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

                        QString filePath("AsusClientTestsLogs");
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

## Static Code Analyse

- Cppchecker
- qtp
