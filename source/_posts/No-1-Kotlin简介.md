---
title: No.1 Kotlin简介
date: 2020-05-14 16:12:41
tags: Kotlin
toc: false
---

`Kotlin` 是一种在 `Java` 虚拟机上运行的静态类型编程语言，被称之为 `Android `世界的`Swift`，由 `JetBrains `设计开发并开源。

`Kotlin` 可以编译成`Java`字节码，也可以编译成 `JavaScript`，方便在没有` JVM` 的设备上运行。

在`Google I/O 2017`中，`Google` 宣布` Kotlin` 成为 `Android `官方开发语言。

<!--more-->

## 1.1  第一个 `Kotlin` 程序

`Kotlin` 程序文件以 **`.kt`** 结尾，如：`hello.kt `、`app.kt`。

```kotlin
package hello                      //  可选的包头
 
fun main(args: Array<String>) {    // 包级可见的函数，接受一个字符串数组作为参数
   println("Hello World!")         // 分号可以省略
}
```

```kotlin
class Greeter(val name: String) {
   fun greet() { 
      println("Hello, $name")
   }
}
 
fun main(args: Array<String>) {
   Greeter("World!").greet()          // 创建一个对象不用 new 关键字
}
```

## 1.2  为什么选择 `Kotlin`？

`Kotlin` 非常适合开发服务器端应用程序，可以让你编写简明且表现力强的代码， 同时保持与现有基于` Java `的技术栈的完全兼容性以及平滑的学习曲线：

- **表现力**：`Kotlin` 的革新式语言功能，例如支持[类型安全的构建器](https://www.kotlincn.net/docs/reference/type-safe-builders.html)和[委托属性](https://www.kotlincn.net/docs/reference/delegated-properties.html)，有助于构建强大而易于使用的抽象。
- **可伸缩性**：`Kotlin `对[协程](https://www.kotlincn.net/docs/reference/coroutines.html)的支持有助于构建服务器端应用程序， 伸缩到适度的硬件要求以应对大量的客户端。
- **互操作性**：`Kotlin` 与所有基于` Java` 的框架完全兼容，可以让你保持熟悉的技术栈，同时获得更现代化语言的优势。
- **迁移**：`Kotlin` 支持大型代码库从 `Java` 到 `Kotlin `逐步迁移。你可以开始用 `Kotlin `编写新代码，同时系统中较旧部分继续用` Java`。
- **工具**：除了很棒的` IDE `支持之外，`Kotlin` 还为` IntelliJ IDEA Ultimate `的插件提供了框架特定的工具（例如 `Spring`）。
- **学习曲线**：对于 `Java` 开发人员，`Kotlin `入门很容易。包含在 `Kotlin` 插件中的自动` Java` 到 `Kotlin` 的转换器有助于迈出第一步。[Kotlin 心印](https://www.kotlincn.net/docs/tutorials/koans.html) 通过一系列互动练习提供了语言主要功能的指南。