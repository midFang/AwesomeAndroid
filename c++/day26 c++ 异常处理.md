c++ 可以抛出任意异常, 相对于 Java 不一样, 比如

## 定义异常

```c++
class Exception // 自定义异常类
{
public:
    string msg;
public:
    Exception(string msg)
    {
        this->msg = msg;
    }

public:
    const char *what()
    {
        return this->msg.c_str();
    }
};
```

## 抛异常

```c++
void main()
{
    try
    {
        int i = 2;
        if (i == 1) 
        {
            throw i;// 抛出 Int 类型的异常, 可以在下面 catch Int 类型的异常
        }
        if (i == 2)
        {
            throw Exception("111"); // 抛出 string 类型的异常
        }
    }
    catch (int number)
    {
        cout << "捕捉到异常" << number << endl;
    }
    catch (Exception &e) // 自定义的异常类, 注意这里是用引用来接受的(推荐)
    {
        cout << "Exception " << e.what() << endl;
    }
    catch (...) // 可以接受到所有异常
    {
        cout << "其他异常" << endl;
    }
}
```

## 异常类的继承

```c++
class Exception : public out_of_range
{
public:
    Exception(string msg) : out_of_range(msg)
    {
        cout << "构造函数" << endl;
    }
    
    Exception(const Exception &e)
    {
        cout << "拷贝构造函数" << endl;
    }


    ~Exception()
    {
        cout << "析构函数" << endl;
    }
};
```

## 抛异常对象和引用的区别

### 抛异常对象

```c++
void c_method()
{ 
    cout << "抛异常" << endl;
    throw Exception("异常了"); // 抛异常对象
}

void main()
{
    try
    {
        c_method();
    }
    catch (Exception e) // Exception：会多次调用构造函数和析构函数
    {
        cout << "try异常:" << endl;
    }
}

输出：
    抛异常
	构造函数
	拷贝构造函数
	try异常:
	析构函数
	析构函数
```

### 抛指针

```c++
void c_method()
{ 
    cout << "抛异常" << endl;
    throw new Exception("我异常了"); // 抛指针
}

void main()
{
    try
    {
        c_method();
    }
    catch (Exception* e) //  Exception* 创建的对象会被析构，如果使用局部函数或者成员就会是一个野指针
    {
        cout << "try异常:" << e->what() << endl;
        delete e; // 需要 delete
    }
}

输出：
    抛异常
	构造函数
	try异常:我异常了
	析构函数
```

### 抛引用

```c++
void c_method()
{
    cout << "抛异常" << endl;
    throw  Exception("我异常了");
}

void main()
{
    try
    {
        c_method();
    }
    catch (Exception &e) // Exception& 避免了多次创建对象 (最多的使用的-推荐)
    {
         cout << "try异常:" << endl;
    }
}

输出：
    抛异常
	构造函数
	try异常
	析构函数		
```

## 抛异常给 Java 

> 前提是需要在 C++ 中 try 处理后再抛出异常, 不  try 处理, Java 层级处理不了 (会直接奔溃)

```c++
void main1()
{
    try
    {
        c_method();
    }
    catch (Exception &e) 
    { 
        cout << "try异常:" << endl;
        // 异常抛出外层
        jclass exceptionClazz = env->FindClass("java/lang/IllegalArgumentException");
      	env->ThrowNew(exceptionClazz,"native层捕获到异常，向java层抛出");
    }catch (...)
    {
        cout << "其他异常" << endl;
    }
}
```

