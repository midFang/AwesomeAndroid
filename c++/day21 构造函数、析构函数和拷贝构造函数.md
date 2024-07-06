## 构造函数，析构函数和拷贝构造函数说明
- 构造函数：创建对象时自动调用，用于初始化对象
- 析构函数：对象销毁时自动调用，用于执行清理操作，析构函数的名称是在类名前加上一个波浪符（~），并且没有返回类型和参数。析构函数的主要作用是释放对象在生命周期内申请的资源，如动态分配的内存、文件句柄等，以避免资源泄露。
- 拷贝构造函数：使用一个已存在的对象来初始化一个新对象时调用。拷贝构造函数的参数通常是一个对同类型对象的引用。拷贝构造函数的主要作用是允许对象被按值传递、返回或者以值的方式初始化另一个对象，同时确保复制的过程中对象的状态被正确复制。(说白了就是方法调用值传递,而不是传递引用的时候,会调用拷贝构造函数,拷贝出来一个副本出来传递)

```c
class Student
{
private:
    char *name;
    int age;

public:
    void setName(char *name)
    {
        cout << "调用 setName 方法" << endl;
        // 常量指针不能给非常量指针赋值, 只能对内容进行 copy
        this->name = name; // 指针 malloc 的方式可以使用这个,但也只是非常量给非常量指针赋值
        // c++ 11 必须要下面这个代码
        // this->name = new char[strlen(name) + 1]; // +1 用于存储 null 终止符, 不要在这里申请内存
        // 需要申请内存, 因为this.name = name, 这样是在栈里面分配的, 方法执行完成就结束了
        // this->name = (char *)malloc(sizeof(char) * 100); // 和上面 new 差不多
        // strcpy(this->name, name);
        cout << "结束调用 setName 方法" << endl;
    }

    void setAge(int age)
    {
        this->age = age;
    }

    char *getName()
    {
        return this->name;
    }

    int getAge()
    {
        return this->age;
    }

public:
    Student()
    { // 空参数构造函数
        cout << "空参数构造函数" << endl;
    }

    Student(int age)
    {
        cout << "一个参数构造函数 age" << endl;
        this->age = age;
    }

    Student(char *name) : age(0) // 一个参数构造函数, 相当于 this->age = 0
    {
        cout << "一个参数构造函数 name" << endl;
        // this->name = name;
        this->name = (char *)malloc(sizeof(char) * 100);
    }

    Student(char *name, int age)
    {
        cout << "两个参数构造函数" << endl;
        this->name = name;
        this->age = age;
    }

    // 这样只能在 c11 中写
    // Student(char *name) : Student("zhangsan", 3) // 一个参数调用 2 个参数的方法
    // {                                            // 调用顺序是先调用 2个构造函数的方法, 再调用自身, 和 java Supur
    //     cout << "一个参数构造函数" << endl;
    // }

    // 析构函数, 就是在对象回收的时候会被调用, 临终遗言, 和 java 的 finalized 方法一样, java 的 finalized 可以拉活对象 ?
    ~Student()
    {
        // 对 对象进行释放操作
        cout << "析构函数" << endl;
        free(this->name);
        this->name = NULL;
    }

    // 拷贝构造函数, 对象会有一个默认的拷贝构造函数，用来对象之间的赋值, 也就是 = 号, 就会调用拷贝构造函数, 对象的赋值操作, 本质上是浅拷贝
    Student(const Student &stu)
    {
        cout << "拷贝构造函数" << endl;
        // this->name = stu.name;// 浅拷贝, 默认情况下是这样
        // 如果动态开辟内存，一定要采用深拷贝 , 为什么 ?
        this->name = (char *)malloc(sizeof(char) * 100);
        strcpy(this->name, stu.name);
        this->age = stu.age; // 基本数据类型不需要动态申请内存, 因为内存大小是固定的
    }
};
```

## 使用
```c
void main(){
    // 情况一
    // 调用无参数构造函数方法
    Student stu; // 这样是在 栈里面开辟了内存, 和 Student stu = NULL, 是不一样的, 这是指向了一个 NULL 空间地址
    stu.setAge(1);
    /**
     * 输出： 空参数构造函数
     *        析构函数 （这个方法执行完，会调用析构函数，内部会释放资源）
     */

    // 情况二
    // 调用无参数构造方法, 返回的是一个一级指针
    Student *stu = new Student();
    /**
     * 输出： 空参数构造函数 (这里没有调用析构函数，是因为 new 是动态申请的内存，并没有delete 释放)
     */

    // 情况三
    Student stu1(22); // 调用一个参数构造方法
    /**
     * 输出： 一个参数构造函数 age
     *        析构函数
     */

    // 情况四
    Student stu;
    stu.setAge(23);
    // c++ 中需要指定为 const
    const char *str = "12midFang"; // 注意：这里如果是常量指针不能传递到 setName 方法里面，因为常量指针不能给非常量指针赋值
    char str1[] = "midFang"; // 这样字符数组可以直接赋值
    stu.setName(str1);
    /**
     * 输出： 空参数构造函数
     *        调用 setName 方法
     *        结束调用 setName 方法
      *       析构函数
     */


    // 情况五
    // malloc 不会调用构造方法 （注：和 new 的区别）
    Student *stu = (Student *)malloc(sizeof(Student));
    stu->setAge(23);
    const char *name = "xiaowang";
    char nameBuffer[10]; // 假设足够大以容纳字符串
    strcpy(nameBuffer, name); // 拷贝值的方式
    stu->setName(nameBuffer); // 间接传递数据到内部方法
     /**
     * 输出： 
     *      调用 setName 方法
     *      结束调用 setName 方法    
     */

     // 情况六
    // 或者这种方式也是可以的, 非常量指针赋值操作而已, 但是都是栈上面分配的内存
    Student stu;
    stu.setAge(23);
    const char *name = "xiaowang";
    char nameBuffer[10]; // 假设足够大以容纳字符串
    strcpy(nameBuffer, name);
    stu.setName(nameBuffer);
    /**
     * 输出： 
     *       空参数构造函数
     *       调用 setName 方法
     *       结束调用 setName 方法
     *       xiaowang , 23
     *       析构函数  
     */


}
```

## 对象的赋值拷贝
```c
void main2()
{
    char name[] = "wangwu";
    Student *stu = new Student(name, 23); // 对象赋值

    Student *stu2 = stu; // = 等号并不会调用拷贝构造函数

    cout << stu->getAge() << " , " << stu->getName() << endl;
    cout << stu2->getAge() << " , " << stu2->getName() << endl;

    cout << stu << " , " << stu2 << endl;
    /**
     * 输出：
     *      两个参数构造函数
            23 , wangwu
            23 , wangwu
            0x1f1830 , 0x1f1830
     */
}
```

## 拷贝构造函数调用时机
```c
Student getStudent(char *name)
{
    Student stu(name); // 栈 ，方法执行完，这个对象会被回收，但是发现调用了拷贝构造函数（不同编译器情况不一样）
    cout << &stu << endl;
    return stu; // 会返回一个新的 Student 对象，但是这个方法执行完成， 而栈内开辟的 stu 是会被回收 （不同编译器情况不一样） 同时这里也是值传递返回回去, stu 是栈对象,也会调用拷贝构造函数返回回去
}

void printStudent(Student stu) // 调用拷贝构造函数, 拷贝一个对象出来, 主要指栈中
{                              // stu 是该方法栈中一个新的对象，拷贝构造函数赋值，方法执行完会调用析构函数
    cout << stu.getName() << " , " << stu.getAge() << endl;
}

void main()
{

    char *name = (char *)malloc(sizeof(char) * 100);
    strcpy(name, "lisi");

    // 第一种场景 作为参数返回的时候会调用拷贝构造函数, 实际运行发现, 这里不会调用拷贝构造函数了，应该是编译器优化了

    /**
     * 值得注意的是C++11引入了移动语义和RVO（返回值优化），
     * 编译器在很多情况下会进行优化以避免不必要的拷贝。
     * 例如，在getStudent函数返回时，它可能仅仅将内存地址移到新的位置，
     * 而不实际进行复制，从而避免调用拷贝构造函数。这依赖于具体的编译器以及其优化设置。
     */
     Student stu = getStudent(name);
     cout << &stu << endl;
    /**
     * 输出：
     *      一个参数构造函数 name
            0x61fdc0
            0x61fdc0
            析构函数
     */

    // 第二种场景 作为参数传递的时候会调用拷贝构造函数
    Student stu(name, 24);
    printStudent(stu);

    /**
     * 输出：
            两个参数构造函数
            拷贝构造函数
            lisi , 24
            析构函数
            析构函数
     * /

    /**
     * 这里回答上面的问题：
     * 如果动态开辟内存，一定要采用深拷贝 , 为什么 ?
     * 
     * 首先这里的 name 就是动态开辟内存的，然后注意到这里（第二种场景）输出调用了 2 次析构函数， 
     * 所以，如果是拷贝构造函数中，使用的是浅拷贝，那么使用的都是同一个对象，那么在
     * 第一次调用析构函数的时候，就已经把name对象释放调用，调用第二次析构就会报错了
     * 
     * 所以如果外部是动态开辟内存的，为了防止多次被调用析构函数，要在析构函数中使用深拷贝，确保是一个新的对象
     * 
     * 注：如果把 char 修改成 string 就不会有这个问题了，因为 string 对内存的管理和释放内部作了处理
     */

}
```

## 总结
- 拷贝构造函数是在值传递的时候, 会拷贝一个对象的副本这个时候会调用拷贝构造函数
- 如果对象内部的属性动态开辟内存，那么拷贝构造函数中一定要采用深拷贝赋值对象, 因为可能会存在悬挂指针的情况, 对象属性在析构函数中被释放了(析构函数被调用了多次)
- 析构函数用于释放内存, 对象生命周期结束时被自动调用, 一般在方法执行完成会调用 (栈内存),如果是动态开辟内存的话,调用 delete 会释放内存, 但是 free 不会调用析构函数,因为 free 只是简单地释放内存，不会处理任何C++对象的构造或析构逻辑, delete 会参数释放内存,和对象的清理,对象拥有的资源等
