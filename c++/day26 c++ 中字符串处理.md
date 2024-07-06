## 对象创建的几种方式

```c++
string str1 = "123";
string str2("123");
string str3(5, 'A'); // 5 个 A = AAAAA
string *str4 = new string("123");  // 指针的创建方式， 需要 delete
```

## 字符串转换

```c++
char* c_str = "fang";
string str(c_str); // 转换成 string 对象
cout << str.c_str() << endl; // 字符串的打印
```

## 字符串遍历


```c++
void main()
{
    string str("hhhfang");

    // 字符串遍历
    for (int i = 0; i < str.length(); i++)
    {
        cout << "遍历方式1：" << str[i] << endl;
    }

    // 字符串遍历
    for (int i = 0; i < str.length(); i++)
    {
        cout << "遍历方式2：" << str.at(i) << endl;
    }

    // 迭代器的方式遍历 (STL标准库)
    for (string::iterator it = str.begin(); it < str.end(); it++)
    {
        cout << "迭代器的方式遍历 " << *it << endl;
    }
}
```

## 字符串添加

```c++
void main()
{
    string str1("123");
    string str2("456");

    string str3 = str1 + str2;
    cout << "str3 " << str3.c_str() << endl;

    string str4 = str1.append(str2).append("dfs");

    cout << "str4 " << str4.c_str() << endl;
}
```

## 字符串删除

```c++
void main()
{
    // 删除
    string str1 = "123 abc 123 abc 123";

    // 第一个参数：从哪里开始 ； 第二个参数：删除几个（默认值，字符串的结尾）
    str1.erase(0, 3); // erase 英: 抹除

    cout << "str1 " << str1.c_str() << endl;

    // 删除内部的 123
    // size_t 是无符号整数类型
    size_t pos = -1;
    string to_erase = "123";
    while ((pos = str1.find(to_erase)) != -1)
    {
        // 查找到 123 的下标，再进行移除
        str1.erase(pos, to_erase.length());
    }

    std::cout << str1 << std::endl; // 输出： " abc  abc "
}
```

## 字符串替换

```c++
void main()
{

    string str1 = "123 abc 123 abc 123";
    // 第一个参数：从哪里开始
    // 第二个参数：替换几个
    // 第三个参数：替换成谁
    str1.replace(0, 6, "1234");

    cout << str1.c_str() << endl; // 1234c 123 abc 123
}

```

## 字符串查找

```c++
void main()
{

    string str1 = "123 abc 123 abc 123";
    // 第一个参数查找谁，第二个参数从哪里开始
    // int position = str1.find("123",0); // 0
    // 从后面往前面查
    int position = str1.rfind("123"); // 16

    cout << position << endl;
}
```

## 字符串大小写转换

```c++
void main()
{

    string str1 = "AAA abc BBB abc 123";

    // toupper 默认是 int
    // [](unsigned char c){ return std::toupper(c); } 是一个匿名函数
    /***
     *[capture](parameters) -> return_type { function_body }
     *  capture是捕获器，it defines what from the outside of the lambda should be available inside the function body and how. 空捕获器 [] 表示这个 lambda 不捕获任何外部变量。
     *
     *parameters：需要接受的参数列表，与普通函数的参数列表类似。
     *
     *return_type：返回类型，这部分可省略，编译器会自动推导。
     *
     *function_body：包含算法的主体部分。
     *上面的lambda表达式 [](unsigned char c){ return std::toupper(c); } 可以解释获译为：它是一个接受一个unsigned char参数c，将其转换为大写并返回的函数。
     */
    transform(str1.begin(), str1.end(), str1.begin(), [](unsigned char c)
              { return std::toupper(c); });

    cout << "大小写转换" << str1 << endl;
}
```

## 综合使用
>  string str1 = "AAA abc BBB abc 123";  abc 全部替换成 def
```c++
void main6()
{
    string str1 = "AAA abc BBB abc 123 abc 456";

    std::string from = "abc";
    std::string to = "def";

    size_t start_pos = 0;
    while ((start_pos = str1.find(from, start_pos)) != std::string::npos)
    { // find(from, start_pos) 第二个参数表示从哪里开始查找
        str1.replace(start_pos, from.length(), to);
        start_pos += to.length(); // 指针偏移
    }

    std::cout << str1 << std::endl; // Outputs: AAA def BBB def 123 def 456
}

```


