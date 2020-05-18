---
title: No.2 Kotlin基础语法
date: 2020-05-18 18:33:37
tags: Kotlin
toc: true
---

## 2.1 函数定义

```kotlin
fun sum(a: Int, b: Int): Int {   // Int 参数，返回值 Int
    return a + b
}
```

表达式作为函数体，返回类型自动推断：

```kotlin
fun sum(a: Int, b: Int) = a + b

public fun sum(a: Int, b: Int): Int = a + b   // public 方法则必须明确写出返回类型
```

<!--more-->

无返回值的函数(类似`Java`中的`void`)：

```kotlin
fun printSum(a: Int, b: Int): Unit { 
    print(a + b)
}


// 如果是返回 Unit类型，则可以省略(对于public方法也是这样)：
public fun printSum(a: Int, b: Int) { 
    print(a + b)
}
```

### 2.1.1 可变长参数函数

函数的变长参数可以用 **`vararg`** 关键字进行标识：

```kotlin
fun vars(vararg v:Int){
    for(vt in v){
        print(vt)
    }
}

// 测试
fun main(args: Array<String>) {
    vars(1,2,3,4,5)  // 输出12345
}
```

### 2.1.2 `lambda`(匿名函数)

`lambda`表达式使用实例：

```kotlin
// 测试
fun main(args: Array<String>) {
    val sumLambda: (Int, Int) -> Int = {x,y -> x+y}
    println(sumLambda(1,2))  // 输出 3
}
```

## 2.2 定义常量与变量

可变变量定义：`var` 关键字

```kotlin
var <标识符> : <类型> = <初始化值>
```

不可变变量定义：`val `关键字，只能赋值一次的变量(类似Java中final修饰的变量)

```kotlin
val <标识符> : <类型> = <初始化值>
```

常量与变量都可以没有初始化值,但是在引用前必须初始化

编译器支持自动类型判断,即声明时可以不指定类型,由编译器判断。

```kotlin
val a: Int = 1
val b = 1       // 系统自动推断变量类型为Int
val c: Int      // 如果不在声明时初始化则必须提供变量类型
c = 1           // 明确赋值


var x = 5        // 系统自动推断变量类型为Int
x += 1           // 变量可修改
```

## 2.3 注释

`Kotlin `支持单行和多行注释，实例如下：

```kotlin
// 这是一个单行注释

/* 这是一个多行的
   块注释。 */
```

与 `Java `不同, `Kotlin `中的块注释允许嵌套。

## 2.4 字符串模板

`$ `表示一个变量名或者变量值

`$varName` 表示变量值

`${varName.fun()}` 表示变量的方法返回值:

```kotlin
var a = 1
// 模板中的简单名称：
val s1 = "a is $a" 

a = 2
// 模板中的任意表达式：
val s2 = "${s1.replace("is", "was")}, but now is $a"
```

## 2.5 `NULL`检查机制

`Kotlin`的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式，字段后加`!!`像`Java`一样抛出空异常，另一种字段后加`?`可不做处理返回值为 `null`或配合`?:`做空判断处理

```kotlin
//类型后面加?表示可为空
var age: String? = "23" 
//抛出空指针异常
val ages = age!!.toInt()
//不做处理返回 null
val ages1 = age?.toInt()
//age为空返回-1
val ages2 = age?.toInt() ?: -1
```

当一个引用可能为 null 值时, 对应的类型声明必须明确地标记为可为 null。

当 `str` 中的字符串内容不是一个整数时, 返回 `null`:

```kotlin
fun parseInt(str: String): Int? {
  // ...
}
```

以下实例演示如何使用一个返回值可为 null 的函数:

```kotlin
fun main(args: Array<String>) {
  if (args.size < 2) {
    print("Two integers expected")
    return
  }
  val x = parseInt(args[0])
  val y = parseInt(args[1])
  // 直接使用 `x * y` 会导致错误, 因为它们可能为 null.
  if (x != null && y != null) {
    // 在进行过 null 值检查之后, x 和 y 的类型会被自动转换为非 null 变量
    print(x * y)
  }
}
```

## 2.6 类型检测及自动类型转换

我们可以使用 `is` 运算符检测一个表达式是否某类型的一个实例(类似于`Java`中的`instanceof`关键字)。

```kotlin
fun getStringLength(obj: Any): Int? {
  if (obj is String) {
    // 做过类型判断以后，obj会被系统自动转换为String类型
    return obj.length 
  }

  //在这里还有一种方法，与Java中instanceof不同，使用!is
  // if (obj !is String){
  //   // XXX
  // }

  // 这里的obj仍然是Any类型的引用
  return null
}
```

或者

```kotlin
fun getStringLength(obj: Any): Int? {
  if (obj !is String)
    return null
  // 在这个分支中, `obj` 的类型会被自动转换为 `String`
  return obj.length
}
```

甚至还可以

```kotlin
fun getStringLength(obj: Any): Int? {
  // 在 `&&` 运算符的右侧, `obj` 的类型会被自动转换为 `String`
  if (obj is String && obj.length > 0)
    return obj.length
  return null
}
```

## 2.7 区间

区间表达式由具有操作符形式 **`..`** 的 `rangeTo` 函数辅以` in` 和` !in` 形成。

区间是为任何可比较类型定义的，但对于整型原生类型，它有一个优化的实现。以下是使用区间的一些示例:

```kotlin
for (i in 1..4) print(i) // 输出“1234”

for (i in 4..1) print(i) // 什么都不输出

if (i in 1..10) { // 等同于 1 <= i && i <= 10
    println(i)
}

// 使用 step 指定步长
for (i in 1..4 step 2) print(i) // 输出“13”

for (i in 4 downTo 1 step 2) print(i) // 输出“42”


// 使用 until 函数排除结束元素
for (i in 1 until 10) {   // i in [1, 10) 排除了 10
     println(i)
}
```

+ 实例测试

  ```kotlin
  fun main(args: Array<String>) {
      print("循环输出：")
      for (i in 1..4) print(i) // 输出“1234”
      println("\n----------------")
      print("设置步长：")
      for (i in 1..4 step 2) print(i) // 输出“13”
      println("\n----------------")
      print("使用 downTo：")
      for (i in 4 downTo 1 step 2) print(i) // 输出“42”
      println("\n----------------")
      print("使用 until：")
      // 使用 until 函数排除结束元素
      for (i in 1 until 4) {   // i in [1, 4) 排除了 4
          print(i)
      }
      println("\n----------------")
  }
  ```

  输出结果：

  ```kotlin
  循环输出：1234
  ----------------
  设置步长：13
  ----------------
  使用 downTo：42
  ----------------
  使用 until：123
  ----------------
  ```