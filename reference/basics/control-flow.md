# 控制流

## If 表达式

Kotlin 中的 `if` 是一个表达式，具有返回值。Kotlin 中没有三元操作符（condition ? then : else），因为 `if` 完全可以胜任这个角色。

```kotlin
// 传统用法
var max = a 
if (a < b) max = b

// 使用 else 
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}
 
// 用作表达式 
val max = if (a > b) a else b
```

`if` 语句的分支可以是代码块，代码块的值是其中最后一个表达式的值：

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

如果要把 `if` 用作表达式而非语句（例如把它的返回值赋给变量），则 `if` 表达式必须包含 `else` 分支。

`if` 的语法见 [Grammar](https://kotlinlang.org/docs/reference/grammar.html#if)。


## When 表达式

`when` 替代了 Java 或类 C 语言中的 `switch`。其最简单的形式如：

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

`when` 会按顺序将参数与各个分支进行比较，直到找到满足条件的分支。`when` 可以用作表达式或语句：如果用作表达式，则整个表达式的值为满足条件的分支的值；如果用作语句，各分支的值会被忽略。（就像 `if` 一样，`when` 的每个分支也可以是代码块，代码块的值是其中最后一个表达式的值。）

当其他分支都不满足条件时，会计算 `else` 分支。把 `when` 用作表达式时，除非编译器能证明分支条件覆盖到了所有可能的情况，否则一定要有 `else` 分支。

如果对多种情况的处理流程相同，可以使用逗号分隔多个分支条件：

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

可以使用任意表达式（而不仅限于常量）作为分支条件：

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

也可以使用 `in`、`!in` 可以判断一个值是否位于[范围](https://blog.nex3z.com/2017/06/14/kotlin-reference-ranges/)或集合中：

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

<a name="注1返回"></a>
还可以使用 `is`、`!is` 来判断一个值是否为某个特定类型，需要注意的是，由于有[智能转换（Smart Cast）](https://blog.nex3z.com/2017/06/15/kotlin-reference-type-checks-casts/#Smart_Casts)，你可以直接访问该类型的方法和成员，而不必进行额外的检查：

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

[【注 1】](#注1)

`when` 还可以用于替换 `if-else if`。如果没有提供参数，且每个分支的条件都是布尔表达式，则会执行条件为 `true` 的分支：

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

`when` 的语法见 [Grammar](https://kotlinlang.org/docs/reference/grammar.html#when)。


## For 循环

`for` 循环可以迭代任意提供了迭代器的对象，语法如下：

```kotlin
for (item in collection) print(item)
```

循环体可以是代码块：

```kotlin
for (item: Int in ints) {
    // ...
}
```

就像上面提到的，循环可以迭代任意提供了迭代器的对象，即：

- 有一个名为 `iterator()` 的成员或扩展函数，其返回类型：
  - 有一个名为 `next()` 的成员或扩展函数，以及
  - 有一个名为 `hasNext()`、返回值为 `Boolean` 的成员或扩展函数。

以上三个方法都必须标记为 `operator`。

`for` 循环遍历数组时会被编译为基于索引的循环，不会创建迭代器对象。

如果想要以索引遍历数组或列表，可以使用如下的形式：

```kotlin
for (i in array.indices) {
    print(array[i])
}
```

注意类似上面这种“遍历一个范围”的形式也会在编译时被优化，避免产生额外对象。

也可以使用 `withIndex()` 库函数：

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

`for` 的语法见 [Grammar](https://kotlinlang.org/docs/reference/grammar.html#for)。


## While 循环

`while` 和 `do`...`while` 的工作方式没有变化：

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y 在这里是可见的
```

`while` 的语法见 [Grammar](https://kotlinlang.org/docs/reference/grammar.html#while)。


## 循环中的 break 和 continue

Kotlin 支持在循环中使用传统的 `break` 和 `continue` 操作符，详见 [返回和跳转](https://blog.nex3z.com/2017/06/02/kotlin-reference-returns-and-jumps/)。


---
<a name="注1"></a>【注 1】这里通过 `is String` 确定 `x` 为 `String` 后，Kotlin 会自动对 `x` 进行类型转换，`x.startsWith("prefix")` 的调用可以直接进行，不需要显式地把 `x` 转换为 `String`。[【返回】](#注1返回)
