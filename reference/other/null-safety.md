# 空安全

## 可为空类型和非空类型

Kotlin 的类型系统致力于从代码中消除空引用的风险（又名 [The Billion Dollar Mistake](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions)）。

在包括 Java 在内的很多语言中，一个最常见的陷阱就是访问一个空引用的成员，继而导致空引用异常，等效于 Java 中的 `NullPointerException`，缩写为 NPE。

Kotlin 的类型系统旨在从代码中消除 `NullPointerException`，仅有以下情况会导致 NPE：

- 显式调用 `throw NullPointerException()`
- 使用 `!!` 操作符，详见下面的描述
- 由外部的 Java 代码导致
- 初始化中的数据不一致（构造器中未初始化的 `this` 变量被外部使用）

在 Kotlin 中，类型系统会区分可以保存 `null` 的引用（可为空的引用）和不可以保存 `null` 的引用（非空的引用）。举例来说，一个常规的 `String` 类型变量不可持有 `null`：

```kotlin
var a: String = "abc"
a = null // 编译错误
```

为了使用 `null`，我们可以声明一个可为空的字符串，写作 `String?`：

```kotlin
var b: String? = "abc"
b = null // OK
```

现在，如果你调用 `a` 的方法或访问其属性，可以确保不会导致 NPE，可以很安全地使用：

```kotlin
val l = a.length
```

但如果想要访问 `b` 上的同样属性，就不那么安全了，编译器会报告错误：

```kotlin
val l = b.length // 错误: 变量 b 不能为 null
```

有下面几种方法可以让我们访问该属性。


## 条件中检查 null

首先，你可以显式地检查 `b` 是否为 `null`，然后针对两种结果进行分别处理：

```kotlin
val l = if (b != null) b.length else -1
```

编译器会记录你进行过的检查的信息，并允许在 `if` 内部访问 `length`。此外也支持更复杂的情况：

```kotlin
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```

注意此方法仅在 `b` 不可变（Immutable）时有效（即在检查和使用之间没有被修改的本地变量，或者具有支持字段且不可被覆盖的 `val` 型成员），否则 `b` 可能会在检查之后变为 `null`。


## 安全调用

第二个选择是使用安全调用操作符，写作 `?.`：

```kotlin
b?.length
```

如果 `b` 不为 `null` 则返回 `b.length`，否则返回 `null`。该表达式的类型为 `Int?`。

安全调用在链式调用中非常有用。比如所，Bob 作为一个职员，可以被分配给某个部门，该部门领导是另一个职员。为了查询 Bob 部门的领导（如果有的话），可以写为：

```kotlin
bob?.department?.head?.name
```

上面的调用链会在任意一个属性为 `null` 时返回 `null`。

如果想要为非空的值进行特定的操作，可以结合使用安全调用操作符和 [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html)：

```kotlin
val listWithNulls: List<String?> = listOf("A", null)
for (item in listWithNulls) {
     item?.let { println(it) } // 输出 A 并忽略 null
}
```


## Elvis 操作符

对于一个可为空的引用 `r`，我们可以说“如果 `r` 不为空，则使用它，否则使用另外的非空值 `x`”：

```kotlin
val l: Int = if (b != null) b.length else -1
```

上面完整的 `if` 表达式可以使用 Elvis 操作符编写，写作 `?:`：

```kotlin
val l = b?.length ?: -1
```

如果 `?:` 左边的表达式不为空，则 Elvis 操作符返回该表达式；否则返回右边的表达式。注意只有在左边的表达式不为空时，才会计算右边的表达式。

另外需要注意的是，因为在 Kotlin 中 `throw` 和 `return` 也属于表达式，所以它们也可以放在 Elvis 操作符的右边。这一点非常方便，如在检查函数参数时：

```kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```


## !! 操作符

第三种选择专为喜欢 NPE 的人士设计。我们可以使用 `b!!`，它会在 `b` 非空时返回 `b` 的值（如这里是 `String`），或者在 `b` 为空时抛出 NPE：

```kotlin
val l = b!!.length
```

由此，你可以获得你想要的 NPE，但必须显式地请求，NPE 不会出乎意料地出现。


## 安全转换

常规的转换会在对象不属于目标类型时导致 `ClassCastException`。此外还可以使用安全转换，安全转换会在转换失败时返回 `null`：

```kotlin
val aInt: Int? = a as? Int
```


## 可为空类型的集合

如果你有一个可为空类型的集合，想要过滤出非空的元素，可以使用 `filterNotNull`：

```kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```