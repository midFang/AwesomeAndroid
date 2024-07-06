- 一元谓词, 二元谓词
> 一元谓词或者是二元谓词 可以理解为就是回调函数，一元谓词可以理解为没有参数，或者是一个参数，二元谓词理解为2个参数的


- 仿函数
> 将回调函数写在了类中里面了，而写在类里面的好处，可以记录状态


```c++
// 仿函数 - 一元谓词 （仿函数的好处是能够记录状态）
class PrintObj
{
public:
    int count = 0; // 记录状态,打印了多少次

public:
    void operator()(int number)
    {
        cout << number << endl;
        count++;
    }
};
```


```c++
// 普通函数, 也是一元谓词
void printObj(int number) 
{
    // cout是一个输出流对象，用于向标准输出设备（通常是屏幕）发送数据。cout的名称来源于“character output”，意为“字符输出”。它是iostream库的一部分，用于实现控制台的输入输出操作。
    // 使用cout时，通常会配合<<运算符，这个运算符在这里被重载为“插入运算符”，用于将数据插入到输出流中。例如，cout << number << endl;这行代码的意思是将变量number的值插入到cout表示的标准输出流中，然后通过endl插入一个换行符并刷新输出缓冲区，使得输出立即出现在屏幕上。
    cout << number << endl;
}
```

## 使用
```c++
// 一元谓词
void main3()
{
    set<int> set1;
    set1.insert(1);
    set1.insert(3);
    set1.insert(2);

    /**
     * for_each 源码分析: 
     *   template<typename _InputIterator, typename _Function>
    _Function
    for_each(_InputIterator __first, _InputIterator __last, _Function __f) // _Function __f 是一个一元谓词
    {
      // concept requirements
      __glibcxx_function_requires(_InputIteratorConcept<_InputIterator>)
      __glibcxx_requires_valid_range(__first, __last);
      for (; __first != __last; ++__first)
    __f(*__first); // 从源码的这个地方可以得知， 只需要传递一个参数的，
      return __f; // N.B. [alg.foreach] says std::move(f) but it's redundant.
    }
    */

    // for_each 标准库中的万能遍历方法
    for_each(set1.begin(), set1.end(), PrintObj()); // 通过一元谓词进行打印
}
```

## 二元谓词

```c++
struct CompareObj // 仿函数, 也是二元谓词
{
public:
    bool operator()(const string str1, const string str2) const
    {
        return str1 > str2; // 降序排序
    }
};

```

## 二元谓词的使用（元素比较）

```c++
void main4()
{
    //   set<string, less<string>> set1; // less 是一个升序排序
    /**
     *   template<typename _Tp> 看源码，可以看出他是一个仿函数，
    struct less : public binary_function<_Tp, _Tp, bool>
    {
      _GLIBCXX14_CONSTEXPR
      bool // 返回值类型
      operator()(const _Tp& __x, const _Tp& __y) const  // 是一个二元谓词(升序)
      { return __x < __y; }
    };
     */

    set<string, CompareObj> set1;

    set1.insert("a");
    set1.insert("b");
    set1.insert("A");
    set1.insert("H");
    set1.insert("C");
    set1.insert("D");

    for (set<string>::iterator it = set1.begin(); it != set1.end(); it++)
    {
        cout << "元素： " << (*it).c_str() << endl;
    }
    /**
     * 输出
     *      元素： b
            元素： a
            元素： H
            元素： D
            元素： C
            元素： A
     */
}
```

## 类中自定义仿函数对象

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
    cout << "count = " << count << endl;

    count = count_if(vector1.begin(), vector1.end(), bind2nd(equal_to<int>(), 2));

    cout << "count = " << count << endl;
}
```
