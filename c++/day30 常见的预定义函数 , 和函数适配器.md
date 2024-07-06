## 字符串的拼接

```c++
void main()
{
    // 字符串的拼接
    plus<string> strAdd; // 查看内部的源码可以得知, 只能两个两个一起添加
    string str = strAdd("aaa", "bbbb");

    cout << "strAdd = " << str << endl;

    set<string, greater<string>> set1; // 降序排序
    set1.insert("aaa");
    set1.insert("bbb");
    set1.insert("ccc");

    for (set<string>::iterator it = set1.begin(); it != set1.end(); it++)
    {
        cout << "set iterator= " << *it << endl;
    }

    // std::find_if 是一个算法，用于在给定的范围内查找满足特定条件的元素。这是一个模板函数，可以用于任何种类的容器，如 vector、list、set 等。
    // std::equal_to 是一个预定义的函数对象（也称为仿函数），它表示等于（==）运算符。std::equal_to 的默认行为和使用 == 运算符是一样的。举例来说，std::equal_to()(5,5) 就会返回 true，因为 5 等于 5。
    // std::bind2nd 是一个函数适配器，它接受一个双参数的函数对象，并"绑定"第二个参数到一个固定的值，然后返回一个单参数的函数对象。在你的例子中，bind2nd(equal_to(), "aaa") 就会返回一个函数对象，这个函数对象接受一个 string 参数，然后比较这个参数和 "aaa" 是否相等。
    // 但是你需要知道的是，std::bind2nd 和 std::equal_to 在 C++11 之后已经被弃用了，它们的功能可以被 lambda 表达式和 std::bind 取代。你的代码可以改写为：

    // 查找集合中的 aaa 字符 (没有像 Java 那种的默认方法)
    set<string>::iterator find_it = find_if(set1.begin(), set1.end(), bind2nd(equal_to<string>(), "aaa"));

    // 使用表达式的方式 std::find_if(set1.begin(), set1.end(), [](const std::string& str){ return str == "aaa"; });

    // TODO 函数适配器 和 lambda 表达式是什么 ? 原理,具体怎么使用的

    if (find_it != set1.end())
    {
        cout << "find_it == " << find_it->c_str() << endl;
    }
    else
    {
        cout << "未找到" << endl;
    }
}
```

## 找出元素个数

```c++
// 1，另外一种自定义仿函数（函数对象）,可以预先传入数据
class Equal
{
private:
    int equal_number;

public:
    Equal(int equal_number)
    { // 传入需要比较的对象
        this->equal_number = equal_number;
    }

public:
    bool operator()(const int &number) // 重载括号运算符
    {
        return number == equal_number;
    }
};

void main1()
{
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(2);
    vector1.push_back(3);
    vector1.push_back(2);
    vector1.push_back(4);
    vector1.push_back(2);

    // 找出集合中， 等于2的个数
    auto count = count_if(vector1.begin(), vector1.end(), Equal(2));
    cout << "count = " << count << endl;  // 输出: 3

    count = count_if(vector1.begin(), vector1.end(), bind2nd(equal_to<int>(), 2));

    cout << "count = " << count << endl;// 输出: 3
}
```

## transform 转换

```c++
// 定义一个函数对象（仿函数）
struct Print
{
    void operator()(int x) const
    {
        std::cout << x << std::endl;
    }
};

int transform_print(int number)
{
    // cout << number << endl;
    return number + 3;
}

void main2()
{
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(2);
    vector1.push_back(3);
    vector1.push_back(2);
    vector1.push_back(4);
    vector1.push_back(2);

    // // 使用 lambda 表达式
    // for_each(vector1.begin(), vector1.end(), [](int number)
    //          { print(number); });

    // for_each(vector1.begin(), vector1.end(), print); 报错了 ，普通函数不行了，只能定义防函数

    for_each(vector1.begin(), vector1.end(), Print()); // for_each 可以遍历所有的集合

    cout << "遍历结束" << endl;

    // resize
    // transform 转换
    vector<int> v2;
    v2.resize(vector1.size());

    transform(vector1.begin(), vector1.end(), v2.begin(), transform_print);

    for_each(v2.begin(), v2.end(), Print());

    /**
     *  1
        2
        3
        2
        4
        2
        遍历结束
        4
        5
        6
        5
        7
        5
     */ 
}
```

## 查找

```c++
void main3()
{
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(2);
    vector1.push_back(3);
    vector1.push_back(2);

    auto find_it = find(vector1.begin(), vector1.end(), 3); // 内部其实调用的是 find_if

    if (find_it != vector1.end())
    {
        cout << "查到的数1： " << (*find_it) << endl;
    }
    else
    {
        cout << "未查到到" << endl;
    }

    // find_if 第三个参数也可以使用仿函数
    auto find_if_it = find_if(vector1.begin(), vector1.end(), [](int number)
                              { return number == 2; });

    if (find_if_it != vector1.end())
    {
        cout << "查到的数2： " << (*find_if_it) << endl;
    }
    else
    {
        cout << "未查到到" << endl;
    }
    /**
     *  查到的数1： 3
        查到的数2： 2
     */
}
```

## 统计

```c++
void main()
{
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(2);
    vector1.push_back(3);
    vector1.push_back(2);
    vector1.push_back(4);

    int number = count(vector1.begin(), vector1.end(), 2); // count 内部使用的其实 count——if 方法，也可以使用 lambda 表达式

    cout << "等于2的个数:" << number << endl;

    number = count_if(vector1.begin(), vector1.end(), [](int number)
                      { return number == 2; });

    cout << "等于2的个数: lambda = " << number << endl;

    number = count_if(vector1.begin(), vector1.end(), bind2nd(less<int>(), 2)); // bind2nd 到底是什么 ？

    cout << "小于2的个数:" << number << endl;

    number = count_if(vector1.begin(), vector1.end(), bind2nd(greater<int>(), 2));

    cout << "大于2的个数:" << number << endl;
    /**
     *  等于2的个数:2
        等于2的个数: lambda = 2
        小于2的个数:1
        大于2的个数:2
     */
}
```

## 合并

```c++
void main()
{
    // 两个有序数组进行合并 - 归并排序
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(2);
    vector1.push_back(3);

    vector<int> vector2;
    vector1.push_back(4);
    vector1.push_back(5);
    vector1.push_back(6);

    vector<int> vector3;
    vector3.resize(vector1.size() + vector2.size());
    merge(vector1.begin(), vector1.end(), vector2.begin(), vector2.end(), vector3.begin());
    for_each(vector3.begin(), vector3.end(), print);
}

/**
 *  number = 1
    number = 2
    number = 3
    number = 4
    number = 5
    number = 6
 */
```

## random_shuffle

```c++
void main()
{
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(3);
    vector1.push_back(2);
    vector1.push_back(4);

    sort(vector1.begin(), vector1.end(), less<int>());
    for_each(vector1.begin(), vector1.end(), print);

    // 打乱循序
    random_shuffle(vector1.begin(), vector1.end());
    for_each(vector1.begin(), vector1.end(), print);
}
```

## 复制和替换

```c++
void main7()
{
    vector<int> vector1;
    vector1.push_back(1);
    vector1.push_back(2);
    vector1.push_back(3);
    vector1.push_back(4);

    vector<int> vector2;
    vector2.resize(2);
    // vector1.begin() ，  vector1.begin() + 2 指针后两个位置， 对应刚刚好是 2 的size大小
    copy(vector1.begin(), vector1.begin() + 2, vector2.begin()); // 一个数组拷贝到另外一个数组中，复制到的是 vector2 数组中
    for_each(vector2.begin(), vector2.end(), print);

    replace(vector1.begin(), vector1.end(), 2, 22); // vector1 数组中的 2，替换为 22
    for_each(vector1.begin(), vector1.end(), print);
}
/**
 *  number = 1
    number = 2
    循环结束
    number = 1
    number = 22
    number = 3
n   umber = 4
 */
```