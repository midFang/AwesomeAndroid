## C++ 中实现多继承
```c
class Person
{
private:
    int age;

public:
    Person(int age)
    {
        this->age = age;
    }

    void setAge(int age)
    {
        this->age = age;
    }

    int getAge()
    {
        return this->age;
    }
};

class Child
{
private:
    char *name;

public:
    Child(char *name)
    {
        this->name = name;
    }
    void setName(char *name)
    {
        this->name = name;
    }

    char *getName()
    {
        return this->name;
    }
};

// 实现多继承, 虽然是多继承, 但是没有实现 （注意，这里并没有二义性）
class Student : public Person, public Child
{
public:
    // 初始化父类中的构造函数
    Student(char *name, int age) : Person(age), Child(name)
    {
    }
};

void main()
{
    Student *stu = new Student("zhangsan", 22);

    stu->setName("adsfa");

    cout << stu->getAge() << endl;
}
```

## 什么是二义性
> 在 java 中不允许多继承, 是因为防止二义性（歧义）的出现, 但是 c++ 是可以多继承的, 但虽然是可以多继承, 但是也不允许有 二义性 的出现

```c
class A
{
private:
    int age;
};

class B : virtual public A
{
};

// class C : public B, public A 这样继承不了,存在二义性
// virtual 加上就可以了, 不加入有二义性, 不知道是继承哪一个类的属性,
// 加上 virtual 确保继承过来的相同属性或者函数，只存在一份拷贝
class C : public B, virtual public A
{
};

void main()
{
    cout << "main1" << endl;

    C *c = new C(); 
}
```

## 什么是虚函数
- 虚函数是一种特殊的成员函数，它在基类中被声明为虚拟的，以允许派生类重写它，实现多态性。虚函数的主要用途是允许在派生类中对基类方法进行特定的实现。


## 多态和抽象类

```c
class Activity // 有 virtual 可以认为是一个抽象类
{
public:
    /**
     * 如果不使用virtual关键字，那么派生类中的函数将无法正确覆盖基类中的函数，而只会隐藏基类中的函数，无法实现多态性
     */
    virtual void onCreate() // 这个函数是一个虚函数。虚函数在面向对象编程中用于实现多态性，允许派生类重写基类中的函数，然后根据实际对象的类型来调用相应的函数
    {
        cout << "Activity 中的 onCreate" << endl;
    }
};

class MainActivity : public Activity
{
public:
    // 父类中, 不加入 virtual 无法覆盖方法
    void onCreate()
    {
        cout << "MainActivity 中的 onCreate" << endl;
    }
};

class WelcomeActivity : public Activity
{
public:
    void onCreate()
    {
        cout << "WelcomeActivity 中的 onCreate" << endl;
    }
};

void startActivity(Activity *aty)
{
    aty->onCreate();
}

void main()
{
    // 多态的使用
    Activity *act1 = new MainActivity();
    Activity *act2 = new WelcomeActivity();

    startActivity(act1);
    startActivity(act2);
}
```

## 
> 虚函数可以用来做什么, 那几个地方会用到
1. 多继承中的二义性
2. 多态
3. 抽象

## 抽象类方法和抽象类

```c
class BaseActivity
{
public: // 定义抽象类方法
    virtual void initView() = 0;
    virtual void initData() = 0;
};

class MyActivity : BaseActivity
{
public:
    // 如果这里没有实现 BaseActivity 中的所有虚函数,则会认为 MyActivity 也是个抽象类, 无法实例化的
    void initView()
    {
        cout << "MyActivity 中的 initView" << endl;
    }
    void initData()
    {
    }
};

void main()
{
    MyActivity *my = new MyActivity();
    my->initView(); 
}
```

## 接口
- 只包含纯虚函数：这些函数没有实现，仅仅定义了一个函数原型。
- 不能直接实例化：接口不能创建对象，它只能被其他类实现。
- 用于定义类之间的协议：接口定义了一个规范，派生类必须实现接口中的所有方法。

```c
// 接口,  如果所有函数都是虚函数, 则可以认为是一个接口 (其实也可以认为是抽象类)
class ClickListener
{
public:
    virtual void click() = 0;
};

// 实现接口
class ImageClickListener : public ClickListener
{
public:
    void click()
    {
        cout << "图片点击" << endl;
    }
};

// 传入指针, 调用指针的方法的
void click(void (*block)()) // / 函数指针作为参数传递  返回值(函数名)(参数)
{
    block();
}

void click1()
{
    cout << "click点击" << endl;
}

void main4()
{
    ImageClickListener *imageListener = new ImageClickListener();

    imageListener->click();

    click(click1); // 函数当作参数传递
    /**
     * 输出：
     *      图片点击    
     *      click点击
     */

}
```

## 模板函数
> 相当于 java 中的泛型

```c
class Math
{
public:
    template <typename T>
    static T max(T a, T b) // 最大值
    {
        return (a > b) ? a : b;
    }
};

// 使用
void main()
{
    std::cout << "Max of 10 and 20 is " << Math::max(10, 20) << std::endl;
    std::cout << "Max of 5.5 and 8.2 is " << Math::max(5.5, 8.2) << std::endl;
    /**
     * 输出
     * Max of 10 and 20 is 20
     * Max of 5.5 and 8.2 is 8.2
    */    
}
```

## 模板类
```c
template <typename T>
class Array
{
private:
    T *data;
    int size;

public:
    Array(int size) : size(size)
    {
        data = new T[size];
    }
    ~Array()
    {
        delete[] data; // 用于释放数组， 不同于 delete（data）这样是释放对象
    }

    // 拷贝构造函数
    Array(const Array &other) : size(other.size), data(new T[other.size]) // 初始化属性
    {
        for (int i = 0; i < size; ++i)
        {
            data[i] = other.data[i];
        }
    }

    void set(int index, T value)
    {
        if (index >= 0 && index < size)
        {
            data[index] = value;
        }
    }
    T get(int index)
    {
        if (index >= 0 && index < size)
        {
            return data[index];
        }
        return T(); // 返回默认构造的对象
    }
};

void main()
{
    // 创建一个整数数组
    Array<int> intArray(5);
    for (int i = 0; i < 5; i++)
    {
        intArray.set(i, i * 2); // 存储数据
    }

    // 输出整数数组的内容
    std::cout << "Integer array contents: ";
    for (int i = 0; i < 5; i++)
    {
        std::cout << intArray.get(i) << " ";
    }
    std::cout << std::endl;

    // 创建一个浮点数数组
    Array<float> floatArray(5);
    for (int i = 0; i < 5; i++)
    {
        floatArray.set(i, i * 1.5f); // 存储数据
    }

    // 输出浮点数数组的内容
    std::cout << "Float array contents: ";
    for (int i = 0; i < 5; i++)
    {
        std::cout << floatArray.get(i) << " ";
    }
    std::cout << std::endl;
}
```

## 拷贝构造函数调用时机
1. 在C++中，如果你定义了一个析构函数，通常也应该考虑定义拷贝构造函数和赋值运算符，这是因为默认的拷贝构造函数只会进行浅拷贝
2. 对于 Array 类，如果使用默认的拷贝构造函数，两个对象将共享相同的数据指针，这可能导致当一个对象被销毁时，另一个对象仍尝试访问已经被释放的内存。这种情况被称为“悬挂指针”。为了避免这种问题，应该实现一个拷贝构造函数来进行深拷贝


## 多参数模板类
```c
template <typename T1, typename T2>
class Pair {
private:
    T1 first;
    T2 second;
public:
    Pair(T1 first, T2 second) : first(first), second(second) {}

    void display() {
        std::cout << "First: " << first << ", Second: " << second << std::endl;
    }
};

void main() {
    // 创建Pair对象，其中包含一个整数和一个浮点数
    Pair<int, float> myPair(123, 456.789);
    myPair.display();

    // 创建Pair对象，其中包含一个字符串和一个整数
    Pair<std::string, int> anotherPair("Hello", 2024);
    anotherPair.display();

    /**
     * 输出:
     *   First: 123, Second: 456.789
     *   First: Hello, Second: 2024
     */
}
```