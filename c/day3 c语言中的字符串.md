## 字符数组和 char* 的区别

```c
void main()
{
    // 字符串的定义， char* 和 char [] 的区别
    // 字符串结尾是 '\0' , 有这个他才知道字符已经结束了

    char str1[] = {'m', 'i', 'd', 'F', 'a', 'n', 'g', '\0'};
    printf("str1 = %s\n", str1);

    // 修改成功
    str1[2] = 'u';
    printf("str1 = %s\n", str1);

    char *str2 = "midFang";

    // 修改失败，char* 的不能修改
    // str2[2] = 'p';
    // *(str2 +2) = 'p';

    printf("str2 = %s\n", str2);
}
```

## 字符串的长度
> 识别 \0 作为结束标识

```c
int str_len_(char *str1)
{
    int len = 0;

    while ((*str1) != '\0')
    {
        len++;
        str1++;
    }

    return len;
}


void main()
{
    char str[] = {'m', 'i', 'd', 'F', 'a', 'n', 'g', '\0', 'i', 's'};

    // 字符长度是 7, \0 是结束符，后面不会再打印了
    printf("字符长度是 %d\n", str_len_(str));

    // 试试官方的， 一样是7
    // printf("字符长度是（官方） %d\n", strlen(str));
}
```

## 字符串的转换

```c
void main()
{
    /**
    atoi()：将字符串转换为整数 ASCII to Integer
    atof()：将字符串转换为浮点数 ASCII to Float
    atol()：将字符串转换为长整型 ASCII to Long
    strtol()：将字符串转换为长整型，支持指定进制 String to Long
    strtoul()：将字符串转换为无符号长整型，支持指定进制 String to Unsigned Long
    strtod()：String to Double
    sscanf()：从字符串中读取格式化数据 String Scanf
    sprintf()：将格式化数据输出到字符串中 String Printf
    **/

    char *str = "12midFang"; // 后面已经被废弃了
    printf("atoi 转换%d\n", atoi(str)); // 转换12, 12 后面的数据会被剔除

    char *str1 = "12.54ffff";
    printf("atof 转换%f\n", atof(str1));               // 12.540000
    printf("strtod 转换%lf\n", strtod(str1, NULL));    // 12.540000
    printf("strtol 转换%ld\n", strtol(str1, NULL, 0)); // 12.540000

    char str3[] = "123 45.6";
    int num;
    float fnum;
    if (sscanf(str3, "%d %f", &num, &fnum) != 2)
    {
        printf("读取失败\n");
    }
    else
    {
        printf("整数：%d,浮点数：%f\n", num, fnum);
    }
}
```

## 字符串比较

```c
void main()
{
    char str1[] = "midFang";
    char str2[] = "midfang";

    // strcmp() 函数在比较字符串时是区分大小写的
    // 不区分大小写的字符串比较 strcasecmp() 或 stricmp() 函数。
    int rc = strcmp(str1, str2); // 不区分大小写比较
    if (rc == 0)
    {
        printf("相等\n");
    }
    else
    {
        printf("不相等\n");
    }
}
```

- strlen()：返回一个字符串的长度（不包括字符串末尾的空字符）
- strcpy()：将一个字符串复制到另一个字符串中
- strncpy()：将一个字符串的一部分复制到另一个字符串中
- strcat()：将一个字符串追加到另一个字符串的末尾
- strncat()：将一个字符串的一部分追加到另一个字符串的末尾
- strcmp()：比较两个字符串是否相等
- strncmp()：比较两个字符串的一部分是否相等
- strstr()：在一个字符串中查找另一个字符串
- strchr()：在一个字符串中查找某个字符第一次出现的位置
- strrchr()：在一个字符串中查找某个字符最后一次出现的位置
- sprintf()：将格式化的数据写入一个字符串中
- strstr() 用于在一个字符串中查找另一个字符串第一次出现的位置

## 字符串查找，包含

```c
void main()
{
    char *str = "my name is wang";
    char *subStr = "wang";
    char *result = strstr(str, subStr); // 返回的是字符出现的指针地址

    if (result == NULL)
    {
        printf("未找到\n");
    }
    else
    {
        int pos = result - str; // result 是字符出现的指针地址，str是字符数组的头指针
        printf("找到了，位置：%s,下标：%d\n", result, pos);
    }
}
```

## 拼接，截取，大小写转换

```c
void main()
{
    char *str = "my name is wang";
    char *str1 = " wu";

    // strcpy 是将一个字符 copy 到 dest 容器中，容器需要有足够大的内存
    char dest[40];
    strcpy(dest, str);
    printf("dest = %s\n", dest); // my name is wang

    // strcat 是连接的意思，将字符复制到末尾
    strcat(dest, str1); // my name is wang wu
    printf("dest = %s\n", dest);

    // 注意不能直接 strcat(str, str1); , 因为c是指针类型的，有可能 str没有足够存放位置了
}
```

## 字符串的截取
```c
char *subString(char *str, int start, int end)
{
    // may snis
    int len = end - start;
    char sub[len + 1];
    // 指针偏移
    str += start;

    for (int i = 0; i < len; i++)
    {
        sub[i] = *str;
        str++;
    }

    sub[len] = '\0';

    char *s = sub;

    return s;
}

// 字符串的截取
void test7()
{
    char *str = "my name is zhang san";
    char *str1 = subString(str, 0, 5);

    printf("subString = %s\n", str1);

    char map[40];
    // 截取
    strncpy(map, str + 0, 5);
    printf("subString = %s\n", map);
}
```


## 大小写转换
```c
void lower(char *dest, char *src, int upper)
{
    while (*src != '\0')
    {
        char ch[1];
        if (upper)
        {
            *ch = toupper(*src);
        }
        else
        {
            *ch = tolower(*src);
        }

        *dest = *ch;
        src++;
        dest++;
    }
    *dest = '\0';
}

//  大小写转换
void main()
{
    char *str = "MY name WANG";
    char dest[50];
    lower(dest, str, 0);

    printf("转换操作：%s\n", dest); // my name wang
}

```

## 替换

```c
char *repleceMethod(char *str, char *src, char *dest)
{
    char *pos = strstr(str, src);
    if (!pos)
    {
        return str;
    }

    int newArraySize = strlen(str) - strlen(src) + strlen(dest) + 1;

    char *result = (char *)malloc(sizeof(char) * newArraySize);
    // 用于将一段内存区域设置为指定的值
    // 将 result 中的每一个字节都设置为了 0
    memset(result, 0, newArraySize);

    int start_pos = pos - str;
    strncpy(result, str, start_pos);
    strcat(result, dest);
    strcat(result, pos + strlen(src));
    result[newArraySize - 1] = '\0';

    return result;
}

// 字符 aaabbcc 要求其中 bb 都替换成 ff
void main()
{
    char str[] = "aaabbcc";
    char *newStr = repleceMethod(str, "bb", "ff");
    printf("dest == %s\n", newStr);  // aaaffcc
    free(newStr);
}
```