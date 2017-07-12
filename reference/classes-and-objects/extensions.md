# 扩展

Kotlin 提供了类似 C# 和 Gosu 中对类进行扩展的功能，可以在不继承类或使用特定设计模式（如修饰者）的情况下，为类添加新的功能。这是通过名为扩展（Extensions）的特殊声明实现的。Kotlin 支持扩展函数和扩展属性。


## 扩展函数

通过在函数名前添加接收者类型（Receiver Type，即被扩展的类型），可以声明扩展函数。如下面为 `MutableList<Int>` 添加了一个 `swap()` 函数：

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // this 对应列表本身
    this[index1] = this[index2]
    this[index2] = tmp
}
```

这里在扩展函数中的 `this` 关键字表示接收者对象（也就是函数名 `.` 前面的类型），之后就可以在任意 `MutableList<Int>` 上调用 `swap()` 了：

```kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // swap() 中的 this 持的是 l 的值
```

当然，这个函数对于任何 `MutableList<T>` 都有用，我们可以使用泛型：

```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

我们在函数名前先声明了泛型参数，这样才能在接收者类型表达式中使用该泛型，详见 [Generic functions](https://blog.nex3z.com/2017/06/08/kotlin-reference-12-generics/#Generic_functions)。


## 扩展是静态解析的

扩展实际上并不会修改被扩展的类。定义扩展时，不会向被扩展的类注入新的成员，而只是添加了一个可以在该类型变量上用点标记（`.`）访问的函数。

需要强调的是，扩展函数的分发是**静态**的，也就是说，它们并不是其接收类型的虚函数。这意味着扩展函数的调用取决于调用扩展函数的表达式的类型，而不是表达式在运行时的结果。举例来说：

```kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

上面的例子会输出 `c`，因为调用哪个扩展函数取决于参数 `c` 的声明类型，也就是类 `C`。

　　如果为类定义的扩展函数和其已有的成员函数具有相同的名称，且都适用于给定参数，则**成员总会胜出**，例如：

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```

在 `C` 的任意实例 `c` 上调用 `c.foo()`，会输出 `member` 而不是 `extension`。

　　但扩展函数可以重载类的成员函数，即具有相同的函数名和不同的签名：

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }
```

调用 `C().foo(1)` 会输出 `extension`。


## 可为空的接收者

注意扩展可以定义在可为空（Nullable）的接收类型上，此类扩展可以在为 `null` 的对象上调用，然后在扩展体内进行 `this == null` 的检查。这也是 Kotlin 允许在不进行 `null` 检查的情况下调用 `toString()` 的原因：检查发生在扩展函数内。

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 在 null 检查之后，this 会自动转换为非空类型，所以下面的 toString()
    // 可以解析为 Any 类的成员函数
    return toString()
}
```


## 扩展属性

类似于扩展函数，Kotlin 也支持扩展属性：

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

注意由于扩展不会向被扩展的类插入成员，并不能让扩展属性具有支持字段（[Backing Field](https://blog.nex3z.com/2017/06/04/kotlin-reference-6-properties-fields/#Backing_Fields)），所以扩展属性不能有初始化，只能通过显式提供 getter 和 setter 来定义扩展属性的行为。

例子：

```kotlin
val Foo.bar = 1 // 错误: 扩展属性不能有初始化
```


## 伴生对象的扩展

如果一个类定义了伴生对象（[Companion Object](https://blog.nex3z.com/2017/06/10/kotlin-reference-15-object-expressions-declarations/#Companion_Objects)），则也可以为伴生对象定义扩展函数和扩展属性：

```kotlin
class MyClass {
    companion object { }  // 将被称为 "Companion"
}

fun MyClass.Companion.foo() {
    // ...
}
```

只需使用类名来调用伴生对象扩展，就像调用伴生对象的普通成员一样：

```kotlin
MyClass.foo()
```


## 扩展的作用域

多数情况下我们把扩展定义在顶层，即直接在包里面：

```kotlin
package foo.bar
 
fun Baz.goo() { ... } 
```

如果要在声明扩展的包外使用该扩展，需要在调用处导入：

```kotlin
package com.example.usage

import foo.bar.goo // importing all extensions by name "goo"
                   // or
import foo.bar.*   // importing everything from "foo.bar"

fun usage(baz: Baz) {
    baz.goo()
}

```

更多信息请参考[导入](https://blog.nex3z.com/2017/06/01/kotlin-reference-packages/#Imports)。


## 声明扩展为成员

可以在一个类内部为另一个类声明扩展，在这种情况下，扩展具有多个隐式的接收者——扩展可以直接访问这些接收者的成员而不必使用限定符（Qualifier）。扩展声明所在的类的实例称为分发接收者（Dispatch Receiver），扩展方法的接收者类型的实例称为扩展接收者（Extension Receiver）。

```kotlin
class D {
    fun bar() { ... }
}

class C {
    fun baz() { ... }

    fun D.foo() {
        bar()   // 调用 D.bar
        baz()   // 调用 C.baz
    }

    fun caller(d: D) {
        d.foo() // 调用扩展函数
    }
}
```

如果分发接收者和扩展接收者具有成员出现命名冲突，则扩展接收者优先，此时如果要访问分发接收者的成员，需要使用 [限定的 `this` 语法](https://blog.nex3z.com/2017/06/19/kotlin-reference-expression/)。

```kotlin
class C {
    fun D.foo() {
        toString()         // 调用 D.toString()
        this@C.toString()  // 调用 C.toString()
    }
```

声明为成员的扩展可以声明为 `open` 并被子类覆盖，这意味着此类扩展函数的分发对于分发接收者类型是虚拟的，但对于扩展接收者是静态的。

```kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())   // 输出 "D.foo in C"
C1().caller(D())  // 输出 "D.foo in C1" - 分发接受者是静态解析的
C().caller(D1())  // 输出 "D.foo in C" - 扩展接受者是动态解析的
```


## 动机

在 Java 中，我们经常使用如 `FileUtils`、`StringUtils` 等以 “*Utils” 的命名工具类，最著名的要数 `java.util.Collections`。工具类不方便的地方在于调用的代码会写成这样：

```java
// Java
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list))
```

这些类名总是很碍事，使用静态导入可以得到：

```java
// Java
swap(list, binarySearch(list, max(otherList)), max(list))
```

这样稍微好一点，但牺牲了 IDE 强大的的代码完成功能。如果能像这样调用是最好的：

```java
// Java
list.swap(list.binarySearch(otherList.max()), list.max())
```

但我们又不想实现 `List` 类中的全部方法，这时就是扩展大显身手的时候了。
