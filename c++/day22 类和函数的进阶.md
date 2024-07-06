## 可变参数

```c
// 可变参数喊出
//  count 这里表示有多少个数需要相加的
// ... 这表示函数可以接受不定数量的参数
int sum(int count, ...) 
{

    va_list vp; // 用于存储可变参数列表信息
    int sum = 0; // 赋初始值

    va_start(vp, count); // 访问可变参数函数参数信息，存储在 vp 中， count 代表读取多少个

    // 循环读取多个参数
    for (int i = 0; i < count; i++)
    {
        // va_arg ：访问下一个可变参数函数参数
        sum += va_arg(vp, int); // type 最后一个参数为类型
    }

    // 释放内存， 释放 vp_list 资源
    va_end(vp); 

    return sum;
}

void main()
{
    int result = sum(2, 1, 8);

    cout << result << endl;
}
```

## 类的静态属性和静态方法
- 可以直接用类名去操作 ::
- 静态的属性必须要初始化 （实现）
- 静态的方法只能去操作静态的属性或者方法


```c
class Person
{
public:
    Person(char *name)
    {
        cout << "构造函数" << endl;
        this->name = name;
    }

    static int age;
    char *name;

public:
    void setName(char *name)
    {
        this->name = name;
    }

    static void changeAge()
    {
        age += 12;
    }
};

// 注：静态属性必须这样初始化, 才不会报错
int Person::age = 1;

/**
 * 静态 可以直接用类名去操作 ::
 *      静态的属性必须要初始化 （实现）
 *      静态的方法只能去操作静态的属性或者方法
 */
void main()
{
    char name[] = "zhangsan";
    Person p(name); // 初始化

    p.age += 11; 
    cout << p.name << ", " << Person::age << endl;
    Person::age += 12; // 也可以直接通过类名来调用

    Person::changeAge(); // 通过类名 :: 调用

    cout << p.name << ", " << Person::age << endl;

    // 输出：
    // 构造函数
    // zhangsan, 12
    // zhangsan, 36
}
```

## 对象大小的计算
> 对象大小的计算, 和结构体的计算是一样的, 同时改变顺序会影响大小

```c
class A
{
public:
    double b; // 8
    int a;    // 4
    char c;   // 1
    static int d; // 静态成员，不计入类大小

    // 2. static 静态变量和方法并没有算到类的大小中
    // 3. 栈，堆，全局（静态，常量，字符串），代码区 ，类的大小只与普通属性有关系

    // 13 , 不能够被8整除, 所以是16
};

void main()
{
    // 对象大小的计算
    int sizeA = sizeof(A);
    cout << "sizeA " << sizeA << endl; // 16
}
```

## const 常量修饰符和, 对 this 的影响
> 当一个成员函数被const修饰时，这表示该函数是一个常量成员函数，它承诺不会修改对象的任何成员变量
```c
class Student
{
public:
    int age;
    char *name;

public:
    // 没有加入 const 可以访问 this 对象
    // this 指针：代表当前的对象，因为类的方法存放在代码区，大家一起共享的，所以要有 this 做区分
    void change()
    {
        cout << "change() " << endl;
        this->age = 44;
        this->name = "23热热阿尔";
    }

    void change1() const // 被 const 修饰后，无法访问 this 
    {
        // 无法访问
        // this->age = 44;
        // this->name = "23热热阿尔";
    }
};

// const 常量修饰符和, 对 this 的影响
void main()
{
    Student *s1 = new Student();

    s1->change();

    cout << s1->age << endl;
    cout << s1->name << endl;
}
```

## 友元函数
> 对于普通函数, 类的内部私有属性只能是类的内部才可以访问, 外部无法访问, 但是被声明称友元函数的时候, 这个友元函数是可以访问的

```c
class Student2
{
private:
    int age;

public:
    void innerChangeAge()
    {
        age = 89;
    };

    void printfAge()
    {
        cout << this->age << endl;
    }

    // 将外部函数声明为友元函数
    friend void outChangeAge(Student2 *stu2);
};

// 外部函数, 访问内部的 private 属性
void outChangeAge(Student2 *stu2)
{
    stu2->age = 333; 
}

void main()
{
    Student2 *s2 = new Student2();

    // 内部函数修改
    // s2->innerChangeAge();
    // s2->printfAge();

    // 友元函数访问修改 private属性
    outChangeAge(s2);
    s2->printfAge();

    delete (s2);
}
```

## 友元类

```c
class ImageView
{
public:
    friend class ImageViewUtil; // 声明友元类

private:
    int a;
};

class ImageViewUtil
{
public:
    void setImage(ImageView image, int obj)
    {
        // 访问ImageView,的 a, 无法访问, 需要声明友元类
        image.a = obj;
    }
};
```