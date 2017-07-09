# 集合

Kotlin 会区分可变（Mutable）和不可变（Immutable）集合（如 List、Set、Map 等），这一点与其他语言不同。通过对集合能否被修改进行精确的控制，有助于消除 Bug 和设计优良的 API。

首先，需要理解可变集合的只读**视图**（View）和一个真正不可变集合间的区别。二者都很容易创建，但在类型系统中并不会体现出区别，你可以根据实际需要来决定是否对二者进行区分。

Kotlin 的 `List<out T>` 类型是一个提供了如 `size`、`get` 等只读操作的接口。它继承自 `Collection<T>`，继而继承自 `Iterable<T>`，就像 Java。

`MutableList<T>` 加入修改列表的方法。`Set<out T>` / `MutableSet<T>` 和 `Map<K, out V>` / `MutableMap<K, V>` 都使用了这种模式。

下面展示了 List 和 Set 类型的基本用法：

```kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // 输出 "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // 输出 "[1, 2, 3, 4]"
readOnlyView.clear()    // -> 无法编译

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```

Kotlin 并没有创建 List 和 Set 的专用语法，使用标准库提供的如 `listOf()`、 `mutableListOf()`、 `setOf()`、 `mutableSetOf()` 等方法即可。如果对性能没有严格要求，可以使用这种简单的[惯用方式](https://kotlinlang.org/docs/reference/idioms.html#read-only-map)来创建映射：`mapOf(a to b, c to d)`。

注意上面代码中的 `readOnlyView` 变量指向的是与 `numbers` 相同的列表，会随着底层列表的改变而改变。如果某个列表只被只读变量引用，则可以认为该列表是完全不可变的，此类列表可以简单地创建如下：

```kotlin
val items = listOf(1, 2, 3)
```

目前 `listOf` 方法是使用一个数组列表实现的，但未来该方法会利用集合不能被修改的事实（`items` 使用 `val` 声明为只读），返回更加节约内存的完全不可变集合类型。

注意只读类型是[协变](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/generics.md#变型)的，这意味着如果 `Rectangle` 继承自 `Shape`， 就可以把 `List<Rectangle>` 赋给 `List<Shape>`；但这对于可变集合类型是不允许的，因为可能会在运行时导致错误。

有时候，你可能希望向调用者返回集合在某一时刻的一个快照，这个快照是确保不会改变的：

```kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```

`toList()` 扩展方法会复制列表中的元素，由此确保返回的列表不会改变。 

List 和 Set 有很多有用的扩展方法：

```kotlin
val items = listOf(1, 2, 3, 4)
items.first() == 1
items.last() == 4
items.filter { it % 2 == 0 }   // 返回 [2, 4]

val rwList = mutableListOf(1, 2, 3)
rwList.requireNoNulls()        // 返回 [1, 2, 3]
if (rwList.none { it > 6 }) println("No items above 6")  // 输出 "No items above 6"
val item = rwList.firstOrNull()
```

此外还有众多工具方法，如 `sort`、`zip`、`fold`、`reduce` 等等。

映射也遵守同样的模式，可以像下面一样简单地实例化和访问：

```kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])  // 输出 "1"
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```
