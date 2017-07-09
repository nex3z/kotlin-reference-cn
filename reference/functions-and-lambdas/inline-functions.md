# 内联函数

使用[高阶函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md)会引入一定的运行时损耗：每一个函数都是一个对象，并且要获取闭包，即在函数内访问外部函数作用域的变量。（对函数对象和类的）内存分配和虚调用都会引入运行时开销。

但在很多情况下，使用内联 Lambda 表达式可以消除此种开销，下面给出的函数很好地展现了这一情况，即 `lock()` 函数可以很容易地在调用处内联，考虑下面的场景：

```kotlin
lock(l) { foo() }
```

编译器不会为该参数创建一个函数对象再进行调用，而是会生成如下的代码：

```kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```

这正是我们一开始想要的结果。

为了让编译器能实现这样的效果，需要使用 `inline` 修饰符标记 `lock()` 函数：

```kotlin
inline fun lock<T>(lock: Lock, body: () -> T): T {
    // ...
}
```

`inline` 修饰符会影响函数本身和传递给该函数的 Lambda：它们都会在调用处内联。

内联会导致生成的代码量变大，但如果合理地利用（不要内联大函数），就会带来性能上的收益，尤其是在循环内的“超级多态（Megamorphic）”调用处。


## 停用内联

对于传递给内联函数的多个 Lambda 表达式，如果只希望内联其中一部分，可以使用 `noinline` 修饰符标记不希望内联的函数参数。

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ...
}
```

内联 Lambda 只能在内联函数内调用，或作为内联参数传递。而对 `noinline` 的 Lambda 则可以进行任意操作，如存储为变量并传递到别处。

注意如果内联函数没有可内联的函数参数，也没有[具体化类型参数](#具体化类型参数)，则编译器会给出警告，因为内联该函数很可能不会带来收益（如果确定需要内联，可以忽略该警告）。


## 非局部返回

在 Kotlin 中，只能使用普通的、未限定的 `return` 来退出一个有名称的函数或匿名函数。这意味着如果想要退出 Lambda，就必须使用[标签](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/basics/returns-and-jumps.md#标签处返回)。禁止在 Lambda 中使用不带标签的 `return`，因为 Lambda 不能让外部的函数返回：

```kotlin
fun foo() {
    ordinaryFunction {
        return // ERROR: 此处不能让 foo 返回
    }
}
```

但如果 Lambda 传递到的函数是内联的，则 `return` 也可以内联，所以允许使用 `return`：

```kotlin
fun foo() {
    inlineFunction {
        return // OK: Lambda 是内联的
    }
}
```

此类返回（位于 Lambda 中，但退出的是外部函数）称为*非局部*（Non-Local）返回。这种结构在包含内联函数的循环中很常见：

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // 令 hasZeros 返回
    }
    return false
}
```

注意有些内联函数不会直接在函数体内调用通过参数传递给它的 Lambda，而是在另一个执行环境中调用，如局部 object 或嵌套函数。在此种情况下，不允许在 Lambda 中使用非局部控制流（Non-Local Control Flow）。为了标识这种情况，Lambda 参数需要使用 `crossinline` 修饰符标记：

```kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```

内联 Lambda 还不支持 `break` 和 `continue`，但我们有支持它们的计划。

 
## 具体化类型参数

有时候需要访问以参数的形式传递的类型：

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```

这里我们沿着树向上，使用反射检查节点是否有一个特定的类型。这没有任何问题，但在调用处却不太好看：

```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

我们实际需要的是把一个类型传递给函数，像这样：

```kotlin
treeNode.findParentOfType<MyTreeNode>()
```

为了达到上面的效果，内联函数支持*具体化类型参数*（Reified Type Parameter），可以这样写：

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

这里使用 `reified` 修饰符限定了类型参数，然后它就可以在函数内被访问了，几乎就像是一个普通的类。由于函数是内联的，不需要使用反射，可以使用如 `!is` 和 `as` 等普通的操作符。当然，也可以像前面提到的那样调用这个函数：`myTree.findParentOfType<MyTreeNodeType>()`。

虽然多数情况下并不需要用到反射，反射仍可以和具体化类型参数一同使用：

```kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```

普通函数（没有标记为内联）不能拥有具体化类型参数，没有运行时表示（Run-Time Representation）的类（如非具体化类型参数，或如 `Nothing` 的虚构类）不能用于具体化类型参数。

更底层的描述，请参考[规范文档](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md)。


## 内联属性（起自 1.1）

`inline` 修饰符可以用于没有支持字段（Backing Field）的属性的访问器，可以标注单个属性访问器：

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

也可以标注整个属性，这样会让两个访问器都成为内联：

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

在调用处，内联属性具有和普通内联函数一样的内联方式。
