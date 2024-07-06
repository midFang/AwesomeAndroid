## 一个 JNI 方法的构成

```c
    external fun changeName() // 在 kotlin 中调用修改名称方法
```

## 在 c 中实现方法
```c
/**
 * extern C: extern "C" 告诉编译器该函数应该以 C 的方式进行编译和链接，而不是使用 C++ 的规则, 作用是告诉编译器在外部链接时需要按照 C 的规则进行处理，以便与其他语言（如 Java）进行交互。
 * 因为在 c里面是不允许函数重载的, 而对于 C++ 函数，默认情况下，编译器会进行名称修饰（name mangling）以支持函数重载和命名空间等特性，这会导致函数名在编译后发生变化。而 Java 使用 JNI 调用 C 或 C++ 函数时(确保函数是唯一的)，需要通过函数名来进行绑定和调用，因此必须使用 extern "C" 来避免名称修饰。
 * fun print(int a)  // 函数重载在 c里面是不行的(生成不了解决方案)
 * fun print(string a)
 *
 *
 * JNIEXPORT JNI 一个关键字，标记为该方法可以被外部调用
 * JNICALL: 也是一个关键字，可以少的 jni call
 * jstring : 代表 java 中的 String
 * jobject: java传递下来的对象，就是本项目中 Simple1 java 对象
 * jclass: java传递下来的 class 对象，就是本项目中的 Simple1.class
 * JNIEnv 这个是 c 和 java 相互调用的桥梁
 * void: 表示没有返回值
 * Java_com_midFang_ndksimple_simple1_Simple1_changeName: 代表是一个 JNI函数: 规则 Java+包名+方法名
 */
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_changeName(JNIEnv *env, jobject thiz) {
    ... 
}
```

## 修改 Java 中的属性
```c
class Simple1 {

    var name: String = "zhangsan"
    external fun changeName()
}

extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_changeName(JNIEnv *env, jobject thiz) {
    // 获取 java 中的class对象
    jclass clazz = env->GetObjectClass(thiz);

    // 获取Java中name的值是什么
    jfieldID jfieldId = env->GetFieldID(clazz, "name", "Ljava/lang/String;");
    jstring j_str = static_cast<jstring>(env->GetObjectField(thiz, jfieldId));
    // j_str 是java中的对象, 需要转换成 c 里面的才能输出打印
    char *oldName = const_cast<char *>(env->GetStringUTFChars(j_str, nullptr));

    printf("name == %s", oldName);

    // Ljava/lang/String; 方法的签名
    jfieldID j_fid = env->GetFieldID(clazz, "name", "Ljava/lang/String;");

    // jstring 是 Java 中的对象
    jstring name = env->NewStringUTF("xiaozhang"); // 创建 java 中的 string 对象
    // 修改name的值,
    env->SetObjectField(thiz, j_fid, name);

    // 内存释放, 用于释放 getStringUTFChars 的字符
    // ReleaseStringChars 用于释放 StringChars 获取的字符 什么样的东西需要释放
    env->ReleaseStringUTFChars(j_str, oldName);
//    env->ReleaseStringChars()
}
```

## 修改 Java 中的静态常量
```c
class Simple1 {
     companion object {
        var age: Int = 24
     }
     external fun changeAge()
}

extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_changeAge(JNIEnv *env, jobject thiz) {
    // 修改 java中的静态属性
    jclass j_clz = env->GetObjectClass(thiz);
    jfieldID jfieldId = env->GetStaticFieldID(j_clz, "age", "I"); // 获取静态常量， 有目的的从后往前写，需要什么数据再获取，更简单

    jint newAge = env->GetStaticIntField(j_clz, jfieldId); // 创建一个 java 中的int
    // c 和 java 中的 newAge 不需要转换
    newAge = 35;

    env->SetStaticIntField(j_clz, jfieldId, newAge);
}
```


## c 调用 java 中的静态方法
```c
class Simple1 {
    companion object {
         //  C 调用静态 java 方法
        external fun callStaticMethod()
        
        fun sum(a: Int, b: Int) {
            println("a + b = ${a + b}")
        }
    }
}

extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_00024Companion_callStaticMethod(JNIEnv *env,
                                                                           jobject thiz) {
    // c 调用 java 中静态的方法

    // thiz 传入的是 simple1:class
    jclass j_clz = env->GetObjectClass(thiz);
    jmethodID j_mid = env->GetMethodID(j_clz, "sum", "(II)V");
    env->CallVoidMethod(thiz, j_mid, 4, 5);
}
```

## c 里面获取签名秘钥
> 打出来的包是 so，放在 c 里面不容易被破解
```c
class Simple1 {
    // 获取签名秘钥
    external fun getSingnaturePassword(): String
}

extern "C"
JNIEXPORT jstring JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_getSingnaturePassword(JNIEnv *env, jobject thiz) {

    return env->NewStringUTF("123");
}
```

## c 调用 java 中的普通实例方法
```c
class Simple1 {
    external fun callSubMethod()
}

extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_callSubMethod(JNIEnv *env, jobject thiz) {
    // c 调用 java 中的普通实例方法

    // thiz 传入的是 simple1 的实例对象（注意)


    jclass jclz = env->GetObjectClass(thiz);
    jmethodID jmid = env->GetMethodID(jclz, "sum", "(II)V");
    // 和 Java_com_midFang_ndksimple_simple1_Simple1_00024Companion_callStaticMethod 基本一致, 只是传入的 thiz 对象不一样
    env->CallVoidMethod(thiz, jmid, 1, 4); // 调用方法
}
```

## c层调用 java 中的静态方法获取 UUID

```c
class Simple1 {
    companion object {
        // 在 c 里面获取 uuid 是比较困难的, 可以在java层级写好, 通过 jni调用
        fun getUUID() = UUID.randomUUID().toString()
    }

    external fun getUUIDJava(): String
}

extern "C"
JNIEXPORT jstring JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_getUUIDJava(JNIEnv *env, jobject thiz) {
    // 实例对象调用静态方法, 获取 uuid

    // 以下 getObjectClass 获取的是实例对象, 通过这个 jclass 无法获取到静态方法的

    jclass classSimple1 = env->FindClass("com/midFang/ndksimple/simple1/Simple1");
    if (classSimple1 == NULL) {
        return NULL; // class not found
    }

    // 第二个参数 "Companion" 是你想要获取的静态字段的名字。在你的Kotlin代码中，你有一个名为 Companion 的伴生对象，这就是我们要找的静态字段名。
    // 第3个参数是类型描述符号, 以L开头, 类名中的 “.” 要被 “/” 替换，最后以 “;” 结尾
    jfieldID companionFieldId = env->GetStaticFieldID(classSimple1, "Companion",
                                                      "Lcom/midFang/ndksimple/simple1/Simple1$Companion;");
    if (companionFieldId == NULL) {
        return NULL; // field 'Companion' not found
    }

    // Object 不能使用传递过来的对象, 因为传递过来的是实例对象, 无法通过它找到 getUUID 方法
    jobject companionObject = env->GetStaticObjectField(classSimple1, companionFieldId);
    if (companionObject == NULL) {
        return NULL; // Companion object not found
    }

    jclass companionClass = env->GetObjectClass(companionObject);
    if (companionClass == NULL) {
        return NULL; // Companion class not found
    }

    jmethodID getUUIDMethodID = env->GetMethodID(companionClass, "getUUID", "()Ljava/lang/String;");
    if (getUUIDMethodID == NULL) {
        return NULL; // method 'getUUID' not found
    }

    return static_cast<jstring>(env->CallObjectMethod(companionObject, getUUIDMethodID));
}
```

## c层调用 java 中的静态方法获取 UUID（静态方法之间的调用）

```c
class Simple1 {
    companion object {
        fun getUUID() = UUID.randomUUID().toString()

        external fun getUUID2(): String
    }
}

extern "C"
JNIEXPORT jstring JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_00024Companion_getUUID2(JNIEnv *env, jobject thiz) {
    // thiz 传递的是伴生类对象就会简单很多了(和上面的区别是这里的对象是伴生对象)
    jclass jclass1 = env->GetObjectClass(thiz);
    jmethodID jmethodId = env->GetMethodID(jclass1, "getUUID", "()Ljava/lang/String;");


    return static_cast<jstring>(env->CallObjectMethod(thiz, jmethodId));
}
```

## jni 构建 java 对象

```c
data class Point(var x: Int, var y: Int)

class Simple1 {
    external fun createJavaObject(): Point // 返回一个 point 对象
}

extern "C"
JNIEXPORT jobject JNICALL
Java_com_midFang_ndksimple_simple1_Simple1_createJavaObject(JNIEnv *env, jobject thiz) {
    // jni 构建 java 对象

    // 需要将.修改成 /
    jclass jclz = env->FindClass("com/midFang/ndksimple/simple1/day12/Point");
    // 获取构造方法
    jmethodID jmethodId = env->GetMethodID(jclz, "<init>", "(II)V");

    jobject point = env->NewObject(jclz, jmethodId, 1, 4); // 通过构造方法创建对象

    /**
     * 修改属性可以有两中方式
     * 1. 通过调用 set 方法修改属性
     * 2. 直接通过属性 修改
     */

    // 直接调用方法修改值
    jmethodId = env->GetMethodID(jclz, "setX", "(I)V");
    env->CallVoidMethod(point, jmethodId, 999);

    // 直接修改属性
    jfieldID jfieldId = env->GetFieldID(jclz, "y", "I");
    env->SetIntField(point, jfieldId, 238);

    return point;
}
```

##  实现 native system.arrayCopy
> 对数组的操作
```c
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_arraycopy(JNIEnv *env, jobject thiz, jobject src, jint src_pos, jobject dest, jint dest_pos, jint length) {
    jclass clazz = env->GetObjectClass(thiz);

    // java 中的 jobject 需要转换成 c 中的对象 jobjectArray
    //  (先获取 env->GetObjectArrayElement()) 得知 需要进行转换
    jobjectArray src_array = reinterpret_cast<jobjectArray>(src);
    jobjectArray dest_array = reinterpret_cast<jobjectArray>(dest);
    __android_log_print(ANDROID_LOG_ERROR, "@@@@ TAG", "开始转换");

    if (src_array != NULL && dest_array != NULL) {
        __android_log_print(ANDROID_LOG_ERROR, "@@@@ TAG", "转换成功");

        for (int i = src_pos; i < src_pos + length; ++i) {
            jobject obj = env->GetObjectArrayElement(src_array, i);
            // 放到新的数组里面
            env->SetObjectArrayElement(dest_array, i, obj);
        }
    }
}

// 使用
private fun test(): Unit {
        val srcArray = arrayOf(Person("张三"), Person("李四"), Person("王武"), Person("小刘"))
        println("@@@@   srcArray ${srcArray.map { it.name }}")
        val destArray = arrayOfNulls<Person>(srcArray.size)
        arraycopy(srcArray, 0, destArray, 0, srcArray.size)
        println("@@@@    destArray  ${destArray.map { it?.name }}")
}

    /**
     * @param src 目标数组
     * @param srcPos 开始的位置
     * @param dest 需要复制的数组
     * @param destPos 开始的位置
     * @param length 长度
     */
    private external fun arraycopy(
        src: Any, srcPos: Int,
        dest: Any, destPos: Int,
        length: Int
    )

```

##  arrayListCopy
> 调用 Java 中的的方法
```c
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_arrayListcopy(JNIEnv *env, jobject thiz, jobject src, jint src_pos, jobject dest, jint dest_pos, jint length) {

    // 获取ArrayList类的引用
    jclass arrayListClass = env->FindClass("java/util/ArrayList");
    jclass destListClass = env->FindClass("java/util/ArrayList");

    // 获取ArrayList类的get和add方法的id
    jmethodID getMethodID = env->GetMethodID(arrayListClass, "get", "(I)Ljava/lang/Object;");
    jmethodID addMethodID = env->GetMethodID(destListClass, "add", "(ILjava/lang/Object;)V");

    // 获取ArrayList类的size方法的id
    jmethodID sizeMethodID = env->GetMethodID(arrayListClass, "size", "()I");

    // 获取srcList的大小
    jint size = env->CallIntMethod((jobject) src, sizeMethodID);

    printf("size = %d ", size);
    __android_log_print(ANDROID_LOG_ERROR, "@@@@ TAG 大小 ", "转换成功 %d ", size);

    /**
     * 一般情况下, 都是从后往前推, 不然不知道调用什么
     * 比如, 这里复制, 需要先遍历 for 循环, 再调用 get 函数, 获取对象, 这样 get 函数需要数据, 就能一步一步往前推导了
     * 然后再是调用 add 添加数据
     */

    // 将srcList中的元素添加到destList中
    for (jint i = 0; i < size; i++) {
        jobject obj = env->CallObjectMethod((jobject) src, getMethodID, i);
        // 赋值到新的数组中
        env->CallVoidMethod((jobject) dest, addMethodID, i, obj);
    }
}

    private external fun arrayListcopy(
        src: Any, srcPos: Int,
        dest: Any, destPos: Int,
        length: Int
    )

// 使用
    private fun ArrayListCopyTest() {
        val srcArray = arrayListOf(Person("张三"), Person("李四"), Person("王武"), Person("小刘"))

        println("@@@@ ArrayListCopy   srcArray ${srcArray.map { it.name }}")

        val destArray = arrayListOf<Person>()

        arrayListcopy(srcArray, 0, destArray, 0, srcArray.size)

        println("@@@@ ArrayListCopy   destArray  ${destArray.map { it?.name }}")
    }
```