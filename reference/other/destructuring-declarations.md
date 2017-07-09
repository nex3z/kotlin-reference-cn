# 解构声明

有时候需要把一个对象*解构*（Destructure）为一系列变量，如：

```kotlin
val (name, age) = person
```

上面的语法称为*解构声明*（Destructuring Declaration），解构声明可以一次性创建多个变量。我们声明了两个变量：`name` 和 `age`，可以单独地使用它们：

```kotlin
println(name)
println(age)
```

解构声明会编译为如下的代码：

```kotlin
val name = person.component1()
val age = person.component2()
```

这里的 `component1()` 和 `component2()` 函数是在 Kotlin 中广泛使用另一个的约定规则（其他约定如 `+` 和 `*` 操作符、`for` 循环等）。只要对象提供了必要数量的 component 函数，该对象就可以用于解构声明（放在解构声明的右边）。当然，也可以存在 `component3()`、`component4()` 等方法。

注意必须使用 `operator` 关键词标记 `componentN()` 函数，这样该函数才能用于解构声明。

解构声明也可以用于 `for` 循环，当使用：

```kotlin
for ((a, b) in collection) { ... }
```

变量 `a` 和 `b` 的值是在集合元素上调用 `component1()` 和 `component2()` 的返回值。


## 例子: 从一个函数中返回两个值

假设我们想从一个函数中返回两个值，比如一个结果对象和一个状态值，在 Kotlin 中可以声明一个[数据类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/data-classes.md)并返回其实例，十分简洁：

```kotlin
data class Result(val result: Int, val status: Status)
fun function(...): Result {
    // 相关计算
    
    return Result(result, status)
}

// 现在可以这样使用该函数：
val (result, status) = function(...)
```

由于数据类会自动声明 `componentN()` 函数，故可以使用解构声明。

**注意**：虽然我们也可以使用标准类 `Pair` 并让 `function()` 返回 `Pair<Int, Status>`，但对数据进行合适的命名往往会更好。


## 例子: 解构声明和映射

遍历一个映射的最好方法可能是：

```kotlin
for ((key, value) in map) {
   // 使用键和值进行相关操作
}
```

要让上面的代码能够工作，我们应当：

- 通过提供 `iterator()` 函数，把 map 表示为一连串的值，
- 通过提供 `component1()` 和 `component2()` 函数，把映射中的元素表示为成对的形式。

实际上，标准库提供了这些扩展：

```kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```

所以我们可以对 `for` 循环中的 map 进行任意的解构（也适用于数据类实例的集合等）。


## 使用下划线替代未使用的变量（起自 1.1）

如果不需要某个解构声明中的变量，可以使用下划线来代替名称。

```kotlin
val (_, status) = getResult()
```


## Lambda 中的解构（起自 1.1）

解构声明的语法可以用于 Lambda 的参数。如果 Lambda 的某个参数是 `Pair` 型（或 `Map.Entry`，或者其他有适当 `componentN` 函数的类型）的，则该参数就可以变为放在括号中的多个参数：

```kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```

注意区分声明两个参数和声明一个参数的解构对的区别：

```kotlin
{ a -> ... } // 一个参数
{ a, b -> ... } // 两个参数
{ (a, b) -> ... } // 一个解构对
{ (a, b), c -> ... } // 一个解构对和另一个参数
```

如果没有用到解构参数的某个成分，则可以使用下划线替代，避免为其命名：

```kotlin
map.mapValues { (_, value) -> "$value!" }
```

你可以指定整个解构的参数的类型，也可以单独指定其某一个成分的类型：

```kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }

map.mapValues { (_, value: String) -> "$value!" }
```
