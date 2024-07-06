## 局部引用 LocalRef
```kotlin
private fun test() {
         // 局部引用
        localRef()
}
```

```c
/** 局部引用， native中创建 java 对象 **/
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_localRef(JNIEnv *env, jobject thiz) {

    // 局部引用的使用
    // 1。 在 native 中， 创建 Java 对象，需不需释放? （需要释放），只有是在 c 层中，创建的对象都记得需要释放掉
    jclass str_clz = env->FindClass("java/lang/String");
    jmethodID jmethodId = env->GetMethodID(str_clz, "<init>", "()V");
    jobject j_str = env->NewObject(str_clz, jmethodId);

    // 局部引用的释放
    env->DeleteLocalRef(j_str);
    // 删除了就不能再使用了，C 和 C++ 都需要自己释放内存（静态开辟的不需要，动态开辟的需要）
}
```

## 全局引用 NewGlobalRef 
> 不同于 LocalRef 的是，全局引用需要在合适的时机手动释放
```kotlin 
private fun test() {
         // 局部引用
        saveGlobalRef("midfang")
                // 获取 nativa 层的全局引用
        val str: String = getGlobalRef()
        println("@@@@ MainActivity  获取 nativa 层的全局引用 $str ")

        // 需要释放全局引用
        deleteGlobal()
        // 输出 midfang
}
```

```c
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_saveGlobalRef(JNIEnv *env, jobject thiz, jstring str) {
    /** 创建全局引用 **/
    globalStr = static_cast<jstring>(env->NewGlobalRef(str));
}

/** 获取全局引用 **/
extern "C"
JNIEXPORT jstring JNICALL
Java_com_midFang_ndksimple_MainActivity_getGlobalRef(JNIEnv *env, jobject thiz) {
    return globalStr;
}

/** 释放全局引用 **/
extern "C"
JNIEXPORT void JNICALL
Java_com_midFang_ndksimple_MainActivity_deleteGlobal(JNIEnv *env, jobject thiz) {
    env->DeleteGlobalRef(globalStr);
}
```

