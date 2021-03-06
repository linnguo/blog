#JNI On Mac - 从Java程序中调用C函数


1.首先在Java中声明本地方法. Java编程语言使用关键字`native`表示本地方法, 本地方法可以是静态的, 也可以是非静态的.

``` java
class HelloNative
{
    public static native void greeting();
}
```
2.使用`javah`工具生成头文件

``` bash
javah HelloNative
```
3.生成`HelloNative.h`, 查看其内容

```
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class HelloNative */

#ifndef _Included_HelloNative
#define _Included_HelloNative
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     HelloNative
 * Method:    greeting
 * Signature: ()V
 */
JNIEXPORT void JNICALL Java_HelloNative_greeting
  (JNIEnv *, jclass);

#ifdef __cplusplus
}
#endif
#endif
```
4.根据生成的接口, 定义函数内容

``` c
#include "HelloNative.h"
#include <stdio.h>

JNIEXPORT void JNICALL Java_HelloNative_greeting(JNIEnv* env, jclass cl)
{
    printf("Hello Native World!\n");
}
```

5.使用C编译器编译C代码

``` bash
export JDK=/System/Library/Frameworks/JavaVM.framework/Versions/A
gcc -dynamiclib \
	-I $JDK/Headers \
	-shared \
	-o libHelloNative.jnilib \
	HelloNative.c
```
6.编写调用代码

``` java
class HelloNativeTest
{
    public static void main(String[] args)
    {
        HelloNative.greeting();
    }

    static
    {
        System.loadLibrary("HelloNative");
    }
}
```
7.运行

``` bash
javac HelloNative.java
javac HelloNativeTest.java
java -Djava.library.path=. HelloNativeTest
```