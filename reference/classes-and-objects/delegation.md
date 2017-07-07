# 委托

## 类委托

实践证明，[委托模式（Delegation Pattern）](https://en.wikipedia.org/wiki/Delegation_pattern)是实现继承的一种有效的替代方式，Kotlin 原生支持委托模式，不需要任何样板代码。下面的代码中，`Derived` 类实现了接口 `Base`，并将其所有的公有方法委托给了一个特定的对象：

```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print() // 输出 10
}
```

`Derived` 类的超类列表中的 `by` 子句表示 `b` 将被保存到 `Derived` 对象的内部，编译器会生成 `Base` 的所有方法并转发给 `b`。

注意覆盖（Override）在这里照常生效，编译器会使用你覆盖的实现，而不是委托对象中的实现，如果我们为 `Derived` 加入 `override fun print() { print("abc") }`，则上面的代码会打印“abc”而不是“10”。
