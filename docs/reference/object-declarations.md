---
type: doc
layout: reference
category: "Syntax"
title: "对象表达式与对象声明"
---

# 对象表达式(Object Expression)与对象声明(Object Declaration)

有时我们需要创建一个对象, 这个对象在某个类的基础上略做修改, 但又不希望仅仅为了这一点点修改就明确地声明一个新类.
Java 通过 *匿名内部类(anonymous inner class)* 来解决这种问题.
Kotlin 使用 *对象表达式(object expression)* 和 *对象声明(object declaration)*, 对这个概念略做了一点泛化.

## 对象表达式(Object expression)

要创建一个继承自某个类(或多个类)的匿名类的对象, 我们需要写这样的代码:

``` kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

如果某个基类有构造器, 那么必须向构造器传递适当的参数.
通过冒号之后的逗号分隔的类型列表, 可以指定多个基类:


``` kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B {...}

val ab: A = object : A(1), B {
    override val y = 15
}
```

如果, 我们 "只需要对象", 而不需要继承任何有价值的基类, 我们可以简单地写:

``` kotlin
val adHoc = object {
    var x: Int = 0
    var y: Int = 0
}
print(adHoc.x + adHoc.y)
```

与 Java 的匿名内部类(anonymous inner class)类似, 对象表达式内的代码可以访问创建这个对象的代码范围内的变量.
(与 Java 不同的是, 被访问的变量不需要被限制为 final 变量.)

``` kotlin
fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0

    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }

        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // ...
}
```

## 对象声明(Object declaration)

[单例模式](http://en.wikipedia.org/wiki/Singleton_pattern) 是一种非常有用的模式, Kotlin (继 Scala 之后) 可以非常便利地声明一个单例:

``` kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```
这样的代码称为一个 *对象声明(object declaration)*, 在 *object*{: .keyword } 关键字之后必须指定对象名称.
与变量声明类似, 对象声明不是一个表达式, 因此不能用在赋值语句的右侧.

要引用这个对象, 我们直接使用它的名称:

``` kotlin
DataProviderManager.registerDataProvider(...)
```

这样的对象也可以指定基类:

``` kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
}
```

**注意**: 对象声明不可以是局部的(也就是说, 不可以直接嵌套在函数之内), 但可以嵌套在另一个对象声明之内, 或者嵌套在另一个非内部类(non-inner class)之内.


### 同伴对象(Companion Object)

一个类内部的对象声明, 可以使用 *companion*{: .keyword } 关键字标记为同伴对象:

``` kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

我们可以直接使用类名称作为限定符来访问同伴对象的成员:

``` kotlin
val instance = MyClass.create()
```

同伴对象的名称可以省略, 如果省略, 则会使用默认名称 `Companion`:

``` kotlin
class MyClass {
    companion object {
    }
}

val x = MyClass.Companion
```

注意, 虽然同伴对象的成员看起来很像其他语言中的类的静态成员(static member), 但在运行时期, 这些成员仍然是真实对象的实例的成员, 它们与静态成员是不同的, 举例来说, 它还可以实现接口:

``` kotlin
interface Factory<T> {
    fun create(): T
}


class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
```

但是, 如果使用 `@JvmStatic` 注解, 你可以让同伴对象的成员在 JVM 上被编译为真正的静态方法(static method)和静态域(static field). 详情请参见 [与 Java 的互操作性](java-to-kotlin-interop.html#static-fields).


### 对象表达式与对象声明在语义上的区别

对象表达式与对象声明在语义上存在一个重要的区别:

* 对象表达式则会在使用处 **立即** 执行(并且初始化)
* 对象声明是 **延迟(lazily)** 初始化的, 只会在首次访问时才会初始化
* 同伴对象会在对应的类被装载(解析)时初始化, 语义上等价于 Java 的静态初始化代码块(static initializer)

