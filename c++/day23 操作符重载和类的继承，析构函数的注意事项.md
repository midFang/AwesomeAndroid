## 操作符重载
```c
class Vector
{
private:
    int x;
    int y;

public:
    Vector(int x, int y)
    {
        this->x = x;
        this->y = y;
    }

    Vector(const Vector &vector)
    {
        this->x = vector.x;
        this->y = vector.y;
        cout << "拷贝构造函数" << endl;
    }

    void setX(int x)
    {
        this->x = x;
    }
    void setY(int y)
    {
        this->y = y;
    }

    int getX()
    {
        return this->x;
    }
    int getY()
    {
        return this->y;
    }

    /**
     * 不使用引用传递的话, 会重复创建对象
     */
    // Vector operator-(const Vector vect) // 到了这里的时候, 会调用拷贝构造函数, 因为main函数的对象是栈中的对象, 进入另外一个方法栈重新拷贝了一个对象
    // {
    //     cout << "operator 方法1 " << endl;
    //     int x = this->x - vect.x;
    //     int y = this->y - vect.y;
    //     cout << "operator 方法2 " << endl;
    //     return Vector(x, y);
    // }

    Vector operator-(const Vector &vect) // 推荐这种(传递)方式
    {
        int x = this->x - vect.x;
        int y = this->y - vect.y;
        return Vector(x, y);
    };

    void operator++(int)
    { // X++
        this->x = this->x + 1;
        this->y = this->y + 1;
    }

    // 模拟打印 << 操作符重载
    friend ostream &operator<<(ostream &_Ostr, const Vector &vector)
    {
        _Ostr << vector.x << "," << vector.y << endl;
        return _Ostr;
    }

    // 条件运算符
    bool operator==(const Vector &vector) // 比较内容
    {
        return (this->x == vector.x && this->y == vector.y);
    }
};


void main()
{
    // 运算符号重载, 加号运算符
    Vector v1(5, 4);
    Vector v2(4, 4);

    Vector v3 = v2 - v1; // Vector 重载了减号操作符，所以可以调用
    v3++;                // Vector 重载了 ++ 操作符，所以可以调用

    // 运算符重载
    cout << "v3 = " << v3 << endl;

    bool isEquals = v1 == v2; // Vector 重载了 == 操作符，所以可以调用

    cout << "isEquals = " << isEquals << endl;
    /*
        输出：
            v3 = 0,1
            isEquals = 0 （bool 非0 就是true，0 代表false）
    */
}
```

## 拷贝构造函数和操作符使用
```c
/**
 * [] 括号运算符重载: 思考, 拷贝构造函数, 析构函数, 什么时候会被调用, 为什么要重写他 ?
 *
 * 写析构函数, 是为了要释放 malloc 申请的对象
 */
class Array
{ // 模仿数组的实现
private:
    int size;
    int *array;

public:
    int operator[](int index)
    {
        return this->array[index];
    }

    Array(int size)
    {
        this->size = size;

        // 申请内存的分配
        this->array = (int *)malloc(sizeof(int) * size);
    }

    // 析构函数
    ~Array()
    {
        cout << "析构函数" << endl;
        if (this->array)
        {
            free(this->array);
            this->array = NULL;
        }
    }

    // 拷贝构造函数, 被析构掉的话, 需要重新创建对象和赋值
    Array(const Array &array)
    {
        cout << "拷贝构造函数" << endl;
        this->size = array.size;

        // 申请内存的分配
        this->array = (int *)malloc(sizeof(int) * array.size);

        // 数据要重新赋值
        for (int i = 0; i < array.size; i++)
        {
            this->array[i] = array.array[i];
        }
    }

    void set(int index, int num)
    {
        array[index] = num;
    }

    int get(int index)
    {
        return array[index];
    }

    int _size()
    {
        return this->size;
    }
};

// notice: 如果这里不是传递的引用, 而是这样传递的, (Array array), 那么这样就会调用析构函数,
// 再调用拷贝构造函数, 会把类中创建的对象给释放掉

/***
 * 为什么会调用析构函数 ?
 *
 * 在C++中，当你传递一个对象给一个函数时，你实际上是把一个对象副本传到函数中，
 * 而不是对象本身。这是由"C++值语义"决定的。也就是说，
 * 函数将会在栈中为传入的对象创建一个新的副本。为了创建这个副本，编译器会调用对象的拷贝构造函数。
 */
void printfArray(Array array) // 这里只是模拟, 相似中需要传递引用, 避免被创建对象了
{
    for (int i = 0; i < array._size(); i++)
    {
        //  cout << array.get(i) << endl;
        //  使用重载运算符打印
        cout << "array [" << i << "]" << array[i] << endl;
    }
}

void mian1()
{
    Array *array = new Array(5);

    // 在一块连续内存上赋值
    array->set(0, 12);
    array->set(1, 13);
    array->set(2, 14);
    array->set(3, 15);

    // 传递的是对象，而不是引用，传递对象到方法里面的时候，会通过拷贝构造函数创建一个对象的副本，而 printfArray 方法执行完成后，又会调用析构函数，所以这里推荐使用引用传递，这里为了演示，模拟这种效果
    printfArray(*array); 

    delete (array); // 释放资源
    /**
     * 输出：
     *  12
        13
        14
        15
        -1163005939 （）因为 array 有5个大小，而只赋值了4个
        析构函数
        析构函数
     */    
}
```

## 类修饰符, 属性初始化

```c
class Person
{
private:
    int age;
    char *name;

public:
    Person(int age, char *name)
    {
        this->age = age;
        this->name = name;
        cout << "父类构造函数, " << endl;
    }

    void setAge(int age)
    {
        this->age = age;
    }

    void setName(char *name)
    {
        this->name = name;
    }

    void test1()
    {
        cout << age << ", " << name << endl;
    }
};

// 类继承修饰符 public
class Student : public Person
{
private:
    char *cource;

public:
    // 可直接初始化父类的构造, 也可初始化本类中的属性
    Student(int age, char *name) : Person(age, name), cource("ss")
    {
        cout << "Student构造函数, " << endl;
    }

    void test2()
    {
        // 调用父类中的方法
        this->setAge(2); 
        test1();
    }
};


int main()
{
    Student stu(34, "zhangsan"); // 栈中开辟内存

    stu.test1(); // 注：类继承不使用 public 修饰符的话, 无法访问
    stu.test2();
    /**
     * 输出：
     *  父类构造函数, 
        Student构造函数,
        34, zhangsan
        2, zhangsan
     */
}
```

## 父类和子类中, 构造函数和析构函数那个先执行
- 创建对象时候, 先调用父类的构造函数, 然后再调用子类的构造函数
- 销毁对象的时候, 先销毁自己(调用析构函数), 再销毁父类的