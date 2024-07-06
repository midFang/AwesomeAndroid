## 局部缓存
```kotlin
    companion object {
        private var name: String? = null
    }

private fun test() {
        staticLocalCache("midfang1")
        println("@@@@ MainActivity 局部缓存1 $name")
        staticLocalCache("midfang22")
        println("@@@@ MainActivity 局部缓存2 $name")
        staticLocalCache("midfang33")
        println("@@@@ MainActivity 局部缓存3 $name")
}

private external fun staticLocalCache(s: String)
```

```c
// 局部静态缓存
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_staticLocalCache(JNIEnv *env, jobject thiz, jstring s) {
    // 给 java 层 name 属性赋值操作 -》需要获取 jfield
    static jfieldID jfieldId = nullptr; //  局部缓存，这个方法会被多次调用，不需要反复的去获取 jfieldID

    // 获取 java 中的class对象（错误）jclass 是对应 java 层的引用，它在函数执行完成，就释放了
    // 和 jfieldId 不一样， jfieldId 是一个针对于 java 的一个属性，它是可以起到局部缓存的效果的，而 jclass 作为一个引用，只能是用全局引用的方式，这样也可以减少调用次数，注意全局引用需要在合适的时机释放
    // GetObjectClass 和 GetStaticFieldID 都涉及到反射的操作，所以它们要这么做
//    static jclass clazz = nullptr;
//    if (clazz == NULL) {
//        clazz = env->GetObjectClass(thiz);
//    } else {
//        LOGI("@@@@ ", "fieldID is not null");
//    }

    if (clazz == NULL) {
        jclass clazz1 = env->GetObjectClass(thiz);
        clazz = static_cast<jclass>(env->NewGlobalRef(clazz1));
    }

    if (jfieldId == NULL) {
        jfieldId = env->GetStaticFieldID(clazz, "name", "Ljava/lang/String;");
        LOGI("@@@@ ", "fieldID is initialized");
    } else {
        LOGI("@@@@ ", "fieldID is not null");
    }
    env->SetStaticObjectField(clazz, jfieldId, s);

    printStaticObjectFieldToLogcat(env, clazz, jfieldId);
}

/** 打印jfield 属性值 **/
void printStaticObjectFieldToLogcat(JNIEnv *env, jclass clazz, jfieldID fieldID) {
    jobject fieldValue = env->GetStaticObjectField(clazz, fieldID);
    const char *fieldValueCStr = NULL;
    if (fieldValue != NULL) {
        fieldValueCStr = env->GetStringUTFChars((jstring) fieldValue, NULL);
        __android_log_print(ANDROID_LOG_INFO, "MyTag", "Field value: %s", fieldValueCStr);
        env->ReleaseStringUTFChars((jstring) fieldValue, fieldValueCStr);
    } else {
        __android_log_print(ANDROID_LOG_INFO, "MyTag", "Field value is null");
    }
}

```

## 输出值
> 需要注意的是 jclass 和 jfield 的处理方式的不同，一个是引用，一个属性，目的都是为了减少查找次数，所做的性能优化
```c
fieldID is initialized
Field value: midfang1
@@@@ MainActivity 局部缓存1 midfang1
fieldID is not null
Field value: midfang22
@@@@ MainActivity 局部缓存2 midfang22
fieldID is not null
Field value: midfang33
@@@@ MainActivity 局部缓存3 midfang33
```

## 全局静态缓存
> 只是将变量变作全局的而已
```kotlin
        /** 全局静态缓存 **/
        initStaticCache()
        setStaicCache("mid-fang")
        println("@@@@ MainActivity  全局静态缓存1 = $name")
        setStaicCache("mid-fang2")
        println("@@@@ MainActivity  全局静态缓存2 = $name")
        setStaicCache("mid-fang3")
        println("@@@@ MainActivity  全局静态缓存3 = $name")

        // 输出
        @@@@ MainActivity  全局静态缓存1 = mid-fang
        @@@@ MainActivity  全局静态缓存2 = mid-fang2
        @@@@ MainActivity  全局静态缓存3 = mid-fang3
```

```c
/** 全局静态缓存 **/
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_initStaticCache(JNIEnv *env, jobject thiz) {
    jclass jclass1 = env->GetObjectClass(thiz);
    // 初始化全局静态缓存
    f_name_id = env->GetStaticFieldID(jclass1, "name", "Ljava/lang/String;");
    f_name1_id = env->GetStaticFieldID(jclass1, "name1", "Ljava/lang/String;");
    f_name2_id = env->GetStaticFieldID(jclass1, "name3", "Ljava/lang/String;");
}


extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_setStaicCache(JNIEnv *env, jobject thiz, jstring name) {
    /** 设置参数 **/
    jclass jclass1 = env->GetObjectClass(thiz); // 优化： 可以采用全局引用
    env->SetStaticObjectField(jclass1, f_name_id, name);
}
```

## Jni 异常的处理方式一，补救措施
> 前置知识，Native 的异常是在 Java 中 catch 不了的

```kotlin
 try {
            exceptionCatch()
            println("@@@@ MainActivity 异常1: name = $name")
        } catch (e: Exception) {
            e.printStackTrace()
        }

        // 输出： 出现异常了
                  @@@@ MainActivity.day15 异常1: name = midFang-nativa


```


```c
/** JNI的异常处理 **/
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_exceptionCatch(JNIEnv *env, jobject thiz) {

    // 模拟尝试给一个不存在的变量赋值
    jclass jclz = env->GetObjectClass(thiz);
    jfieldID f_id = env->GetStaticFieldID(jclz, "name33333", "Ljava/lang/String;");

    /**
     * 第一种方式
     * 检测 Java 是否有异常， 如果有异常，做补救措施， 比如， name33333 不存在， 修改一下，改为查找一个存在的属性
     */
    jthrowable throwable = env->ExceptionOccurred(); // 检测是否有异常
    if (throwable) {
        LOGI("@@@@ ", "出现异常了");
        // 1。清除异常（必须要做的事）不清楚异常，还是会奔溃
        env->ExceptionClear();
        // 2。重新获取 name 属性
        f_id = env->GetStaticFieldID(jclz, "name", "Ljava/lang/String;");
    }

    jstring name = env->NewStringUTF("midFang-nativa");
    env->SetStaticObjectField(jclz, f_id, name);
}
```

## Jni 异常的处理方式二，抛异常给 java 层级

```kotlin 
// Jni 异常的处理，第二种throws给java的方式
        try {
            exceptionCatch2()
            println("@@@@ MainActivity.day15 异常2: name = $name")
        } catch (e: Exception) {
            e.printStackTrace()  // 会被 catch 出异常
        }
```


```c
/** 异常的处理， 抛异常 **/
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_exceptionCatch2(JNIEnv *env, jobject thiz) {
    // 模拟尝试给一个不存在的变量赋值
    jclass jclz = env->GetObjectClass(thiz);
    jfieldID f_id = env->GetStaticFieldID(jclz, "name33333", "Ljava/lang/String;");

    jthrowable throwable = env->ExceptionOccurred(); // 检测是否有异常

    if (throwable) {
        // 清除异常
        env->ExceptionClear();
        // Throw 抛一个 java 的 Throwable 对象
        jclass no_such_clz = env->FindClass("java/lang/NoSuchFieldException");
        env->ThrowNew(no_such_clz, "NoSuchFieldException name3");

        return;// 记得 return, 不 return 执行后续的逻辑，依然是不存在的 属性赋值
    }

    jstring name = env->NewStringUTF("Darren");
    env->SetStaticObjectField(jclz, f_id, name);
}
```