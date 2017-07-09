# 区间

可以使用 `rangeTo()` 函数（对应操作符为 `..`）编写区间表达式（Range Expression），并使用 `in` 和 `!in` 来补充。区间可以用于所有可比较的类型，但对于整型基本类型（Integral Primitive Type）的实现有优化。下面是一些使用区间的例子：

```kotlin
if (i in 1..10) { // equivalent of 1 <= i && i <= 10
    println(i)
}
```

整型区间（`IntRange`、`LongRange`、`CharRange`）有一个额外的功能：它们可以被迭代。编译器会把迭代转换为类似 Java 中基于索引的 `for` 循环的形式，不会带来额外开销。

```kotlin
for (i in 1..4) print(i) // 输出 "1234"

for (i in 4..1) print(i) // 没有输出
```

逆序迭代一组数字也很简单，可以使用标准库中的 `downTo()` 函数：

```kotlin
for (i in 4 downTo 1) print(i) // 输出 "4321"
```

如果想用 1 以外的任意步长来迭代数字，可以使用 `step()` 函数：

```kotlin
for (i in 1..4 step 2) print(i) // 输出 "13"

for (i in 4 downTo 1 step 2) print(i) // 输出 "42"
```

使用 `until` 函数可以创建不包含结尾元素的区间：

```kotlin
for (i in 1 until 10) { // i 位于 [1, 10), 不包含 10
     println(i)
}
```


## 工作原理

区间实现了库中的一个公共接口：`ClosedRange<T>`。

`ClosedRange<T>` 表示数学意义上的闭区间，用于可比较类型。它有两个端点，`start` 和 `endInclusive`，二者都包含在区间内。它主要的操作为 `contains`，通常以 `in` / `!in` 操作符的形式使用。

整型数列（`IntProgression`、`LongProgression`、`CharProgression`）表示数学上的等差数列，数列由首元素（`first`）、尾元素（`last`）和一个非零的步长（`step`）定义。第一个元素是 `first`，后续元素是前一个元素加上 `step`，`last` 元素总会在迭代中出现，除非数列是空的。

数列是 `Iterable<N>` 的子类，这里的 `N` 是 `int`、`Long` 或 `Char`。所以数列可以用于 for 循环和 `map`、`filter` 等函数。对 `Progression` 进行迭代等同于 Java / JavaScript 中的基于索引的 for 循环：

```kotlin
for (int i = first; i != last; i += step) {
  // ...
}
```

对于整型类型，`..` 操作符会创建一个实现了 `ClosedRange<T>` 并继承了 `*Progression` 的对象。比如 `IntRange` 实现了 `ClosedRange<Int>` 并继承了 `IntProgression`，因此适用于 `IntProgression` 的操作也适用于 `IntRange`。`downTo()` 和 `step()` 的结果始终是 `*Progression`。

通过在数列的伴生对象中定义的 `fromClosedRange` 函数来构造数列：

```kotlin
IntProgression.fromClosedRange(start, end, step)
```

数列中的 `last` 元素是计算得到的，满足 `(last - first) % step == 0`：对于正的 `step`，`last` 的值为满足条件的不大于 `end` 的最大值，；对于负的 `step`，`last` 的值为满足条件的不小于 `end` 的最小值。 


## 实用函数

### rangeTo()

`rangeTo()`操作符用于整型类型上时，只是简单地调用了对应 `*Range` 类的构造器，如：

```kotlin
class Int {
    //...
    operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
    //...
    operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
    //...
}
```

浮点类型数字（`Double`、`Float`）没有定义自己 `rangeTo` 操作符，而是使用标准库为泛用 `Comparable` 类型提供的 `rangeTo` 作为替代：

```kotlin
public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

该函数返回的区间不能用于迭代。

### downTo()

对于任意两个整型类型的组合，都有对应的 `downTo()` 扩展函数，如下面的两个例子：

```kotlin
fun Long.downTo(other: Int): LongProgression {
    return LongProgression.fromClosedRange(this, other.toLong(), -1L)
}

fun Byte.downTo(other: Int): IntProgression {
    return IntProgression.fromClosedRange(this.toInt(), other, -1)
}
```

### reversed()

对于每一个 `*Progression` 类，都定义了 `reversed()` 扩展函数，用于返回逆序的数列。

```kotlin
fun IntProgression.reversed(): IntProgression {
    return IntProgression.fromClosedRange(last, first, -step)
}
```

### step()

对于每一个 `*Progression` 类，都定义了 `step()` 扩展函数，用于根据 `step`（函数参数）的值返回具有对应步长的数列。步长值必须为正数，因此这个函数永远不会改变迭代的方向：

```kotlin
fun IntProgression.step(step: Int): IntProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return IntProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return CharProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}
```

需要注意的是，为了保证不变式 `(last - first) % step == 0`，该函数返回数列的 `last` 可能会与原数列的 `last` 不同，例如：

```kotlin
(1..12 step 2).last == 11  // 数列的值为 [1, 3, 5, 7, 9, 11]
(1..12 step 3).last == 10  // 数列的值为 [1, 4, 7, 10]
(1..12 step 4).last == 9   // 数列的值为 [1, 5, 9]
```
