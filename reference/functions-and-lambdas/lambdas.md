# 高阶函数和 Lambda

## 高阶函数

高阶函数指的是以函数作为参数或返回值的函数。`lock()` 函数是一个很好的例子，它接受一个对象和一个函数作为参数，获取对象的锁后，执行函数并释放锁：

```kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    }
    finally {
        lock.unlock()
    }
}
```

来看一下上面的代码：`body` 是[函数类型](#函数类型)`() -> T`，这个函数没有参数，有一个类型为 `T` 的返回值。它被 `lock` 保护，在 `try` 块中被调用，其结果由 `lock()` 函数返回。

如果要调用 `lock()` 函数，可以向其传递另一个函数作为参数（详见[函数引用](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/reflection.md#函数引用)）。

```kotlin
fun toBeSynchronized() = sharedResource.operation()

val result = lock(lock, ::toBeSynchronized)
```

另外，使用 [Lambda 表达式](#lambda-表达式和匿名函数)通常会更加方便：

```kotlin
val result = lock(lock, { sharedResource.operation() })
```

Lambda 表达式的内容详见[下面](#lambda-表达式和匿名函数)，但为了继续说明这一节的内容，我们先来看一下它的其简介：

- Labmda 表达式总要被大括号包围，
- 其参数（如果有的话）在 `->` 的左边声明（参数类型可以省略），
- 函数体（如果有的话）写在 `->` 的右边。

如果函数的最后一个参数是函数类型，且你传递的是 Lambda 表达式，Kotlin 的惯例是把 Lambda 表达式写在括号外面：

```kotlin
lock (lock) {
    sharedResource.operation()
}
```

高阶函数的另一个例子是 `map()`：

```kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
    val result = arrayListOf<R>()
    for (item in this)
        result.add(transform(item))
    return result
}
```

`map()` 的调用方法如下：

```kotlin
val doubled = ints.map { value -> value * 2 }
```

注意如果 Lambda 表达式是函数调用的唯一参数，则可以省略函数调用时的括号。

### it: 单个参数的隐式名称

另一个有用的惯例是，如果字面函数只有一个参数，则可以省略其声明（以及 `->`），此时参数的名称是 `it`：

```kotlin
ints.map { it * 2 }
```

使用这些惯例，可以写出 [LINQ 风格](https://msdn.microsoft.com/en-us/library/bb308959.aspx)的代码：

```kotlin
strings.filter { it.length == 5 }.sortBy { it }.map { it.toUpperCase() }
```

### 使用下划线替代未使用的变量（起自 1.1）

如果没有用到 Lambda 表达式的某个参数，可以使用下划线替代其名称：

```kotlin
map.forEach { _, value -> println("$value!") }
```

### 在 Lambda 中解构（起自 1.1）

在 Lambda 中使用解构的方法详见 [解构声明](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/destructuring-declarations.md#lambda-中的解构起自-11)。


## 内联函数

有时候，使用[内联函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/inline-functions.md)可以提高高阶函数的性能。


## Lambda 表达式和匿名函数

Lambda 表达式和匿名函数属于“函数字面值（Function Literal）”，也就是没有经过声明而直接作为表达式传递的函数，考虑下面的例子：

```kotlin
max(strings, { a, b -> a.length < b.length })
```

函数 `max` 是一个高阶函数，它的第二个参数是一个函数。上面的用法中，`max` 的第二个参数是是一个表达式，该表达式本身是一个函数，即函数字面值；作为一个函数，它等效于：

```kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### 函数类型

对于一个接受其他函数作为参数的函数，我们必须为该参数指定一个函数类型，如前面提到的 `max` 定义如下：

```kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
    var max: T? = null
    for (it in collection)
        if (max == null || less(max, it))
            max = it
    return max
}
```

参数 `less` 的类型是 `(T, T) -> Boolean`，也就是接受两个类型为 `T` 的参数并返回 `Boolean` 的函数：如果第一个参数小于第二个，则返回 `true`。在第 4 行的函数体中，可以像函数一样调用 `less`，传递给它的是两个类型为 `T` 的参数。

除了像上面的书写方式，函数类型也可以有命名参数，可以记录各个参数的含义：

```kotlin
val compare: (x: T, y: T) -> Int = ...
```

### Lambda 表达式语法

Lambda 表达式（即函数类型字面值）的完整句法如下：

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

Lambda 表达式始终被大括号包围。完整句法形式的参数声明写在括号内，且有可选的类型标注。函数体写在 `->` 符号后面。如果推断 Lambda 的返回值不是 `Unit`，则 Lambda 体内最后一个（也可能是唯一一个）表达式的值会作为函数的返回值。

如果省略掉所有的可选标注，剩下的部分如下：

```kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

只有一个参数的 Lambda 表达式十分常见，如果 Kotlin 能够自己判断出签名，则可以把这唯一的参数声明也省略掉，该参数会被隐式地声明为 `it`：

```kotlin
ints.filter { it > 0 } // 该函数字面值的类型为 (it: Int) -> Boolean
```

可以使用[限定的返回](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/basics/returns-and-jumps.md#标签处返回)语法显式地从 Lambda 中返回值，否则 Lambda 中的最后一个表达式会被隐式地返回，因此，下面两段代码是等效的：

```kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

注意如果一个函数以另一个函数作为最后一个参数，则以 Lambda 表达式为该参数时，可以在参数列表的括号外面传递，相关语法详见 [callSuffix](https://kotlinlang.org/docs/reference/grammar.html#callSuffix)。

### 匿名函数

上面所述的 Lambda 表达式语法缺少指定函数返回值类型的能力，在多数情况下，这并没有必要，因为返回值类型可以被自动推断出来。但是如果需要显式地指定返回值类型，可以使用另外一种语法：*匿名函数*（Anonymous Function）。

```kotlin
fun(x: Int, y: Int): Int = x + y
```

匿名函数看上去非常像常规的函数声明，但它省略了函数名。匿名函数的函数体可以是表达式（如上）或代码块：

```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

匿名函数可以像常规的函数一样指定参数和返回值，不同之处在于，如果参数类型可以通过上下文被推断出来，就可以省略参数类型：

```kotlin
ints.filter(fun(item) = item > 0)
```

匿名函数返回值类型的推断和普通函数一样：如果匿名函数的函数体是表达式，则返回值类型会被自动推断；如果匿名函数的函数体是代码块，则必须显式地指明返回值类型（不指定则假定为是 `Unit`）。

注意匿名函数的参数总是在括号内传递，在括号外传递函数的简写语法仅适用于 Lambda 表达式。

匿名函数和 Lambda 表达式的另一个区别是[非局部返回（Non-Local Returns）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/inline-functions.md#非局部返回)的行为。没有标签的 `return` 总是会让使用 `fun` 关键字声明的函数返回，这意味着 Lambda 表达式中的 `return` 会让包围在外部的函数返回，而匿名函数中的 `return` 会让匿名函数本身返回。

### 闭包

Lambda 表达式和匿名函数（以及[局部函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/functions.md#局部函数)和[对象表达式](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/objects.md#对象表达式)）可以访问其*闭包*（Closure），也就是在外部作用域声明的变量。不同于 Java 的是，可以修改从闭包中获取的变量：

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

### 带接收者的函数字面值

Kotlin 提供了在调用函数字面值时指定*接收者对象*（Receiver Object）的能力。在函数字面值内部，可以直接调用接收对象的方法，而不必使用额外的限定符。这类似于扩展函数，扩展函数内部也可以访问接收对象的成员。这一功能最重要的用途之一是实现[类型安全的 Groovy 风格建造者（Type-Safe Groovy-Style Builders）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/type-safe-builders.md)。

此类函数字面值的类型是带有接收者的函数类型：

```kotlin
sum : Int.(other: Int) -> Int
```

可以像调用接收者对象的方法一样调用该函数字面值：

```kotlin
1.sum(2)
```

匿名函数的语法允许直接指定函数字面值的接收对象的类型，这样就可以先声明一个带有接收者的函数变量，以便后续使用：

```kotlin
val sum = fun Int.(other: Int): Int = this + other
```

当接收类型可以从上下文推断出来时，Lambda 表达式也可以用作带接收者的函数字面值。

```kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // 创建接收者对象
    html.init()        // 把接收者对象传递给 Lambda
    return html
}


html {       // 带接收者的 Lambda 从此处开始
    body()   // 调用接收者对象的方法
}
```
