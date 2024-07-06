# 集合

## vector 数组

```c++
void main()
{
    // 创建 vector 数据

    vector<int> v;
    // vector<int> v(10); // 10个元素，内部都是0
    // vector<int> v(10, 2); // 10 个元素，内部都是2

    // 插入元素
    v.insert(v.begin(), 1);
    v.insert(v.begin(), 2); // v.begin() 最开始的位置，此刻打印结果是 2，1

    v.insert(v.end(), 33); // v.end() 最后一个，最开始的位置，此刻打印结果是：2，1，33

    // 迭代
    for (int i = 0; i < v.size(); i++)
    {
        cout << "v = " << v.at(i) << endl;
    }

    cout << "v.front = " << v.front() << endl; // 2
    cout << "v.back = " << v.back() << endl; // 33 

    // 引用当左值和右值的时候，可以修改值
    v.front() = 222; // 最前面的
    v.back() = 333;  // 最后面的

    cout << "v.front = " << v.front() << endl;  // 222
    cout << "v.back = " << v.back() << endl; // 333

    v.push_back(95); // 放入最后一个数据

    // 删除数据
    v.erase(v.begin()); // 通过迭代器函数，删除的是最开始的数据

    // 迭代器的方式
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++)
    {
        cout << "v = " << *it << endl;
    }
}
```

## stack 栈

> 栈，先进后出

```c++
void main()
{
    stack<int> s;

    // 压栈 ，压入指定位置
    s.push(12);
    s.push(22);
    s.push(32);

    // 出栈
    s.pop();

    // 遍历
    while (!s.empty())
    {
        int top = s.top(); // 获取栈顶的值
        cout << "stack : " << top << endl;
        s.pop(); // 出栈
    }

    // 如果想获取 stack 中的某一个元素位置, 应该怎么做 ?
    // 在 java 中是可以做到的, 但是 C++ 做不到, 它只允许用户访问、添加或移除栈顶元素，来满足先进后出的特性
}
```

## queue 队列

> 队列，先进先出

```c++
void main()
{
    queue<int> q;

    // 入队
    q.push(12);
    q.push(44);
    q.push(32);

    // 引用作为左值进行修改
    q.front() = 42;

    cout << q.front() << endl; // 最开始的元素, 打印: 42 

    q.pop(); // 弹出后 元素为 (44, 32)

    cout << q.front() << endl; // 打印: 44

    // 最后，最后加入的
    int back = q.back();
    cout << back << endl; //// 打印:  32

    while (!q.empty())
    {
        cout << q.front() << endl; // 队列是先进先出的结构
        q.pop();
    }
}
```

## priority_queue 优先级队列

> 具有顺序的队列

```c++
void main()
{
    // 参数说明
    // int 存放的数据
    // vector<int> 数据类型（数组）
    // greater 从大到小 less 从小到大
    priority_queue<int, vector<int>, greater<int>> queue; // 内部默认仅仅支持 int 类型的数据,如果是其他结构的数据,需要自定义排序方式

    queue.push(23);
    queue.push(2233);
    queue.push(123);
    queue.push(3);
    queue.push(3);

    while (!queue.empty())
    {
        cout << "quene: " << queue.top() << endl;
        queue.pop();
    }
    // 打印结果: 3, 3, 23, 123, 2233
}
```

## list 双向链表

```c++
void main()
{
    list<int> l;

    // 插入
    l.push_front(11);
    l.push_back(22);
    l.insert(l.begin(), 10); // 元素为: (10, 11, 12)

    // 修改
    l.back() = 33;  // 元素为: (10, 11, 33)
    l.front() = 44; // 元素为: (44, 11, 33)

    // 不能通过角标去访问，也不能去修改

    // 移除
    // l.erase(l.begin());  // 元素为: (11, 33)
    // l.pop_front(); // 移除头节点元素
    // l.pop_back(); // 移除尾节点元素

    // 循环
    for (list<int>::iterator it = l.begin(); it != l.end(); it++)
    {
        cout << "list: " << *it << endl; // 元素为: (44, 11, 33)
    }
}
```

## 函数谓词

> 在C++中，函数谓词（Function Predicate）是指一个返回布尔值（true或false）的函数或函数对象。它通常用于表示某种条件或属性，比如在算法中用于比较、搜索、排序等操作的条件判断。

函数谓词可以是一个普通函数，也可以是一个重载了operator()的类的实例（即函数对象），还可以是一个lambda表达式。它们通常用于标准库中的算法，如std::sort、std::find_if等，以提供自定义的条件逻辑。

- 示例
  
  ```c++
    普通函数作为谓词：
        bool isEven(int n) {
             return n % 2 == 0;
        }
  
    使用函数对象作为谓词：
        struct IsEven {
           bool operator()(int n) const {
             return n % 2 == 0;
        }
  
    使用lambda表达式作为谓词：
        auto isEven = [](int n) { return n % 2 == 0; };
  ```

- 使用:
  
  ```c++
  std::vector<int> vec = {1, 2, 3, 4, 5};
      auto it = std::find_if(vec.begin(), vec.end(), isEven);
      在这个例子中，std::find_if使用isEven谓词来查找第一个偶数。
  ```

## 自定义函数谓词在集合中使用

```c++
class Student
{
public:
    string name;
    int grade;

public:
    Student(string name, int grade)
    {
        this->name = name;
        this->grade = grade;
    }
};

// 函数对象 仿函数
struct CompareFuns // 自定义排序结构
{
    // 函数重载了 () 运算符，函数对象，仿函数
    bool operator()(const Student &_Left, const Student &_Right) const
    {
        return _Left.grade > _Right.grade; // 从大到小排序
    }
};

void main()
{
    // 基本数据类型 ，对象数据类型(本身不支持比较)
    set<Student, CompareFuns> stus; // CompareFuns 是一个自定义排序结构

    Student st1("fang", 12);
    Student st2("wang", 13);
    Student st3("ling", 1);

    stus.insert(st1);
    stus.insert(st2);
    stus.insert(st3);

    for (set<Student>::iterator it = stus.begin(); it != stus.end(); it++)
    {
        cout << "set = " << it->name.c_str() << ", " << it->grade << endl;
    }

    // 输出结果: wang, 13 ,  fang, 12,  ling, 1  (根据 grade 从大到小排序)
}
```

## set 容器
> 红黑树结构 ，会对你存入的数据进行排序，但是不允许元素相同

```c++
void main()
{
    set<int, greater<int>> s;

    // 添加参数 , (区别)不需要用迭代器，也不需要指定位置
    s.insert(3);
    s.insert(5);
    s.insert(4);

    // 判断是否插入成功
    pair<set<int, greater<int>>::iterator, int> pair = s.insert(5); // 元素相同不允许插入

    bool result = pair.second;

    if (result)
    {
        cout << "插入成功,插入元素: " << *pair.first << endl;
    }
    else
    {
        cout << "插入失败" << endl;
    }

    // 遍历
    for (set<int>::iterator it = s.begin(); it != s.end(); it++)
    {
        cout << *it << endl;
    }
}
```
## multiset 容器
> 允许重复 ，用法和 set 一样

```c++
void main7()
{
    multiset<int, greater<int>> ms;

    // 添加参数 , 不需要用迭代器，也不需要指定位置
    ms.insert(3);
    ms.insert(5);
    ms.insert(4);
    ms.insert(4); // 允许重复元素
    ms.insert(3);

    for (set<int>::iterator it = ms.begin(); it != ms.end(); it++)
    {
        cout << *it << endl;
    }
}
```

## map 容器
> 会对 key 排序 ，二叉树算法

```c++
void main()
{
    // map 会对 key 排序 ，二叉树算法
    map<int, string> map;

    // 插入数据的几种方式
    map.insert(pair<int, string>(01, "01"));

    map.insert(make_pair<int, string>(02, "02"));

    auto pair = map.insert(make_pair<int, string>(02, "03")); // auto是C++中的一个关键字，用于自动类型推导

    if (!pair.second)
    {
        cout << "插入失败" << endl;
    }

    map[04] = "04"; // 这种会直接覆盖, 上面的如果是相同的key,会插入失败

    // 查找: find方法接受一个键作为参数，并返回一个指向找到的元素的迭代器。如果没有找到元素，它将返回一个指向map末尾的迭代器（map.end()
    auto find_it = map.find(111);

    if (find_it != map.end())
    {
        //  找到了元素，it->first 是键，it->second 是值
        cout << "找到元素" << find_it->first << ", " << find_it->second << endl;
    }
    else
    {
        cout << "没找到元素" << endl;
    }

    // 需要注意到是 map 集合获取一个不存在的对象, 会插入一个默认的对象并返回(是一个默认的实例),不能像 java 那个 != null 判空
    string find_it2 = map[200]; // 返回的是一个空字符, 如果是 int 类型,则是返回0,如果是对象,返回默认的构造实例
    cout << "find_it2 == " << find_it2 << endl;


    // 以下遍历的几种方式
    for (std::map<int, std::string>::iterator it = map.begin(); it != map.end(); ++it)
    {
        std::cout << it->first << " => " << it->second << '\n';
    }

    for (const auto &pair : map)
    {
        std::cout << pair.first << " => " << pair.second << '\n';
    }

    for (const auto &[key, value] : map)
    {
        std::cout << key << " => " << value << '\n';
    }
    /**
     *  插入失败
        没找到元素
        find_it2 ==
        1 => 01
        2 => 02
        4 => 04
        200 =>
        1 => 01
        2 => 02
        4 => 04
        200 =>
        1 => 01
        2 => 02
        4 => 04
        200 =>
     */ 
}

```

## multimap 
> 可以存储重复的元素

```c++
void main()
{
    // multimap 可以存储重复的元素
    multimap<int, string> map;
    // 案例，1 （11，12，13），2 （21，22，23），3（31，32，33）

    map.insert(pair<int, string>(1, "11"));
    map.insert(pair<int, string>(1, "12"));
    map.insert(pair<int, string>(1, "13"));

    map.insert(pair<int, string>(3, "31"));
    map.insert(pair<int, string>(3, "32"));
    map.insert(pair<int, string>(3, "33"));

    map.insert(pair<int, string>(2, "21"));
    map.insert(pair<int, string>(2, "23"));
    map.insert(pair<int, string>(2, "22"));

    // 迭代
    for (multimap<int, string>::iterator it = map.begin(); it != map.end(); it++)
    {
        cout << "multimap == first = " << it->first << ", second = " << it->second.c_str() << endl;
    }

    cout << " 遍历结束" << endl;

    // 查询 key 为3 的多个数据
    // equal_range方法。这个方法返回一个迭代器对，其中first是指向第一个匹配元素的迭代器，second是指向最后一个匹配元素之后位置的迭代器。通过遍历这个范围，可以访问所有键为3的值。
    auto range = map.equal_range(3);
    for (auto it = range.first; it != range.second; ++it)
    {
        cout << it->second << endl;
    }
    /**
     * 输出结果
     *  multimap == first = 1, second = 11
        multimap == first = 1, second = 12
        multimap == first = 1, second = 13
        multimap == first = 2, second = 21
        multimap == first = 2, second = 23
        multimap == first = 2, second = 22
        multimap == first = 3, second = 31
        multimap == first = 3, second = 32
        multimap == first = 3, second = 33
        遍历结束
        31
        32
        33
     */
}
```

## 集合中存储对象

```c++

class Person
{
private:
    string name;

public:
    Person(string name);
    ~Person();
    string getName();
    void setName(string name);
    Person(const Person &otherPerson);
};

// 以下是类的实现, 只是分开来了

// 构造函数
Person::Person(string name)
{
    this->name = name;
    cout << "构造函数" << endl;
}

string Person::getName()
{
    return name;
}

void Person::setName(string name)
{
    this->name = name;
}

// 析构函数
Person::~Person()
{
    cout << "析构函数" << endl;
}

Person::Person(const Person &otherPerson)
{
    cout << "拷贝构造函数被调用" << endl;
    this->name = otherPerson.name;
}

void main2()
{
    // java 中把对象添加到了集合
    // c++ 中会调用对象的拷贝构造函数，存进去的是另一个对象
    // 第一个错误：没有默认的构造函数
    // 第二个错误：析构函数也可能回调用多次，如果说在析构函数中释放内存，需要在拷贝构造函数中进行深拷贝
    vector<Person> vector1;

    Person p1("fang"); // 调用一次构造函数和调用一次析构函数

    vector1.push_back(p1);

    p1.setName("jack");

    cout << "name 的值1 " << p1.getName() << endl;

    auto v1 = vector1.front(); // 打印的仍然还是 fang, 因为他们两的对象不一样,  c++ 中会调用对象的拷贝构造函数，存进去的是另一个对象， 如果不想这样， 想改变的都是同一个对象，则存储指针就可以了

    cout << "name 的值2 " << v1.getName() << endl;
}

/**
 * 输出结果:
 *          构造函数
            拷贝构造函数被调用
            name 的值1 jack
            拷贝构造函数被调用
            name 的值2 fang
            析构函数
            析构函数
            析构函数
 */
```

## 集合存储对象指针

```c++
vector<Person*> vector1;

Person p1("fang");
vector1.push_back(&p1);

p1.setName("jack");

cout << "name 的值1 " << p1.getName() << endl;
cout << "name 的值2 " << vector1.front()->getName() << endl;

/**
 *  构造函数
    name 的值1 jack
    name 的值2 jack
    析构函数
 */
```