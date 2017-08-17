# 枚举类

枚举类最基础的应用是实现类型安全的枚举：

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

每一个枚举常量都是一个对象，每个枚举常量之间以逗号分隔。


# 1. 初始化

由于每一个枚举都是枚举类的实例，故而它们可以被初始化：

```kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

# 2. 匿名类

枚举常量可以声明自己的匿名类：

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

匿名类可以有自己的方法，也可以覆盖父类的方法。需要注意的是，如果枚举类中定义了成员，就需要使用分号来分隔枚举常量定义和成员定义，和 Java 一样。


# 3.使用枚举常量

Kotlin 也像 Java 一样提供了列出已定义的枚举常量的方法，以及根据名称获取枚举常量的方法，这些方法的签名如下（假设枚举类名为 `EnumClass`）：

```kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

如果 `valueOf()` 不能将指定的名称和枚举类中定义的枚举常量相匹配，则会抛出 `IllegalArgumentException` 异常。

<a name="注1返回"></a>
从 Kotlin 1.1 开始，可以使用 `enumValues<T>()` 和 `enumValueOf<T>()` 函数，以泛型的方式来访问枚举类中的常量：

```kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // prints RED, GREEN, BLUE
```
[【注 1】](#注1)

每个枚举常量都有以下属性，用于获取枚举常量的名称，及其在枚举类中定义的位置：

```kotlin
val name: String
val ordinal: Int
```

枚举常量也实现了 [`Comparable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 接口，比较使用的是常量在枚举类中定义的自然顺序。


---
<a name="注1"></a>【注 1】上面的代码中，`printAllValues<RGB>()` 以泛型的方式传递了枚举类 `RGB`，`printAllValues()` 通过 `enumValues<T>()` 获取了 `RGB` 中定义的所有枚举常量。`reified` 表示[具体化参数类型](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/inline-functions.md#具体化类型参数)。[【返回】](#注1返回)
