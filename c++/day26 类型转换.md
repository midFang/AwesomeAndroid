## static_cast

1. 用于基本数据类型之间的转换，如把 int 转换成 char
2. 把类型转换成另一种类型，用于类层次结构中基类和派生类之间指针或引用的转换

```c++
static_cast<type-name>(expression)
type-name 是你想要转换的目标类型，它可以是任何指针、引用、或者是指针/引用的数值类型。而 expression 是你要被转换的表达式
```

```c++
void main()
{
    double number = 23.4; // doublue 类型转换为 int 类型

    int num1 = static_cast<int>(number);

    cout << "doublue 类型转换为 int 类型 " << num1 << endl; // 23

    int num2 = 44;

    char a = static_cast<int>(num2);

    cout << "int 类型转换为 char 类型 " << a << endl; // 什么都没有打印，也没有报错

    Person *p1 = new Person("fang", 10);

    // 转换失败，父类不能转换为子类
    // Student s = static_cast<Student *>(p1);
    // Student s = p1;

    // 子类转换为父类 （在java中，相当于是多态）
    Student *stu = new Student("fang", 20);

    Person *p2 = static_cast<Person *>(stu);
    cout << "子类转换为父类 " << p2->name.c_str() << endl;

    /**
     * 打印结果：
     * doublue 类型转换为 int 类型23
     * int 类型转换为 char 类型,
     * 子类转换为父类0xf24990
     */
}
```

## const_cast

> 转换:允许你将一个 const 对象转换为非 const 对象 

```c++
void main()
{
    const Person *p = new Person("fang", 30);

    // 通常情况下以下都不能修改，因为 p 是一个常亮
    // p->age = 23;
    // p->name = "fang1";

    // 使用 const_cast 转换为可以修改
    Person *p1 = const_cast<Person *>(p);
    p1->name = "fasgd";

    cout << "子类转换为父类 p1 " << p1->name.c_str() << endl;
    cout << "子类转换为父类 p " << p->name.c_str() << endl;
}
```

## reinterpret_cast

- 用途： `reinterpret_cast`用于进行低级别的类型转换，可以将任何指针类型转换为任何其他指针类型（包括与原始类型无关的类型），甚至可以将指针转换为足够大的整数类型。它也可以用于转换整数类型到指针类型，以及反向操作
- 安全性：这种转换不检查类型的兼容性，因此非常危险，容易导致未定义行为。使用时需要非常小心，确保转换是合理的

```c++
    // 除了字符类，各种类型的转换  long -> 对象的指针* ， 用到 reinterpret_cast
    // 与 static_cast 区别在于 static_cast 一般用于转换有继承关系的类型 reinterpret_cast 也能转换继承关系的类型
    Person *p = new Person("fang", 32);
    Student *stu = reinterpret_cast<Student *>(p); // 父类转换成子类 （和 static_cast 不同，static_cast 做不到）

    cout << "reinterpret_cast 转换 stu " << stu->name.c_str() << endl;
        // long mPtr = (long)p; // 无法转换
    // long mPtr = reinterpret_cast<long>(p); 无法转换
    // Student *stu1 = reinterpret_cast<Student *>(mPtr);

    // cout << "reinterpret_cast 转换 stu1 " << stu1->name.c_str() << e
```

## dynamic_cast

> `dynamic_cast`用于处理对象的向下转换（即从基类指针转换到派生类指针），并且它在运行时检查这种转换是否安全（其他的转换都是编译期会进行检测）。这种转换只有在基类有至少一个虚函数时才可能成功，因为这样的类被认为是多态的，而多态性是`dynamic_cast`正确工作所必需的，如果转换失败，会返回 NULL

```c++
void main()
{
    Person *stu = new Person("fang", 24);

    // 从父类转换为子类的时候，需要保证基类中至少有一个虚函数，编译器才知道这是一个多态类，并且dynamic cast操作的类型必须是多态类型
    Student *p = dynamic_cast<Student *>(stu); // 父类转换成子类 ---》 转换失败

    Student *stu1 = new Student("fang", 24);
    Person *p1 = dynamic_cast<Person *>(stu1); // 子类转换成父类 ---》 转换成功
    if (stu1 != NULL)
    {
        cout << "stu1 转换成功" << endl;
    }
    else
    {
        cout << "stu1 转换失败" << endl;
    }

    // 转换失败会返回 NULL
    if (p != NULL)
    {
        cout << "p 转换成功" << endl;
    }
    else
    {
        cout << "p 转换失败" << endl;
    }
    delete stu;
    // cout << "dynamic_cast 转换 p " << p->name.c_str() << endl;
}
输出： 
    stu1 转换成功
	p 转换失败
```

## 总结

1. static_cast 静态转换  用于基本数据类型之间的转换，如把int转换成char
2. const_cast 常量转换 用于修改常量的值，允许你将一个 const 对象转换为非 const 对象
3. reinterpret_cat 强制类型转换 ，用于转换任意类型
4. dynamic_cast 动态转换 ，更安全，转换成功返回类型，失败返回空 ，
   - 必须要包含多态类型和 static_cast 很类似，但是更安全

- static_cast 可以在完全不相关的类型之间进行转换（例如 int 到 double），也可以在类的层次结构中进行转换
- dynamic_cast 专门用于处理类的层次结构，并且它会在运行时检查是否可以转换