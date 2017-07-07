# 对象表达式和声明

有时候我们想对一个类进行些许修改，而又不想显式地声明一个新的子类。Java 使用*匿名内部类*（Anonymous Inner Class）的方式应对这种场景，Kotlin 使用*对象表达式*（Object Expression）和*对象声明*（Object Declaration）进一步拓展了这个概念。


## 对象表达式

要创建继承了某个（或多个）其他类型的匿名内部类，可以使用如下的方法：

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

如果超类有构造器，则必须为其传递合适的构造器参数。多个超类可以在 `object :` 的冒号后面列出，并以逗号分隔：

```kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B {...}

val ab: A = object : A(1), B {
    override val y = 15
}
```

如果只是需要“一个对象”，而不需要它有超类，则可以简单地写为：

```kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
```

需要注意的是，匿名对象的类型仅在局部和 private 声明中有效，如果使用匿名对象作为 public 函数的返回类型或 public 属性的类型，则该函数和属性的实际类型为匿名对象声明的超类（如果匿名对象没有声明超类，则为 `Any`），此种情况下，加入到匿名对象的成员是不可访问的。

```kotlin
class C {
    // Private function, so the return type is the anonymous object type
    private fun foo() = object {
        val x: String = "x"
    }

    // Public function, so the return type is Any
    fun publicFoo() = object {
        val x: String = "x"
    }

    fun bar() {
        val x1 = foo().x        // Works
        val x2 = publicFoo().x  // ERROR: Unresolved reference 'x'
    }
}
```

对象表达式内部的代码可以访问外部作用域的成员，就像 Java 的匿名内部类一样。（但又不仅限于访问 final 变量，这一点与 Java 不同。）

```kotlin
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


## 对象声明

[单例（Singleton）](https://en.wikipedia.org/wiki/Singleton_pattern) 是一种非常有用的模式，Kotlin（在 Scala之后）让声明单例变得很简单：

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

这种使用 `object` 关键字后加名称的声明，称为对象声明（Object Declaration）。就像变量声明一样，对象声明不是表达式，不能放在赋值语句的右边。

使用名称可以直接引用对象：

```kotlin
DataProviderManager.registerDataProvider(...)
```

这些对象也可以具有超类：

```kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
}
```

**注意**：对象声明不能是局部的（如直接嵌套在函数中），但可以嵌套在其他对象声明或非内部类中。

## 伴生对象

类中的对象声明可以使用 `companion` 关键字标记，成为伴生对象（Companion Object）：

```kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

伴生对象的成员可以简单地通过类名来引用：

```kotlin
val instance = MyClass.create()
```

伴生对象的名称可以省略，此时会使用 `Companion` 作为名称：

```kotlin
class MyClass {
    companion object {
    }
}

val x = MyClass.Companion
```

需要注意的是，虽然伴生对象的成员看上去像是其他语言中的静态成员，但是在运行时，伴生对象的成员仍是实际（单例）对象的实例成员，并且可以实现接口：

```kotlin
interface Factory<T> {
    fun create(): T
}


class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
```

然而在 JVM 上可以使用 `@JvmStatic` 注解，让伴生对象的成员生成为真正的静态方法和字段，更多信息请见 [Java Interoperability](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#static-fields) 一节。

## 对象表达式和对象声明的语义区别

对象表达式和对象声明之间有一个非常重要的语义区别：

- 对象表达式会在使用处**立即**执行（并初始化），
- 对象声明的初始化是**延迟**的，仅在第一次被访问时初始化，
- 伴生对象在相应的类被载入（解析）时初始化，对应 Java 的静态初始化器的机制。
