# 类型别名

使用类型别名可以为已有类型提供了替代的名称。如果某个类型的名称很长，可以为其引入一个更短的名称，以便于使用。

类型别名可以缩短过长的泛型类型，举例来说，通常希望缩短集合类型：

```kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

可以为函数类型提供别名：

```kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```

可以为内部类和嵌套类提供别名：

```kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

类型别名不会引入新的类型，它们等同于对应的原类型。当你添加了 `typealias Predicate<T>` 并在代码中使用 `Predicate<Int>` 时，Kotlin 编译器始终会把它展开为 `(Int) -> Boolean`，因此该类型的变量可以用在任何要求常规函数类型的地方，反之亦然：

```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main(args: Array<String>) {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // 输出 "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // 输出 "[1]"
}
```
