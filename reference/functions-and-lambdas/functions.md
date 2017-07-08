# 函数

## 声明函数

在 Kotlin 中使用 `fun` 关键字定义函数：

```kotlin
fun double(x: Int): Int {
    return 2*x
}
```

## 使用函数

函数的调用和传统方法一样：

```kotlin
val result = double(2)
```

使用点表示法（`.`）调用成员函数：

```kotlin
Sample().foo() // 创建 Sample 类的实例并调用 foo
```

### 中缀表示法

满足以下条件时，也可以使用中缀表示法调用函数：

- 是成员函数或[扩展函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md)
- 只有一个参数
- 使用 `infix` 关键字标记

```kotlin
// 为 Int 定义扩展
infix fun Int.shl(x: Int): Int {
...
}

// 使用中缀表示法调用扩展函数

1 shl 2

// 等同于

1.shl(2)
```

### 参数

函数的参数使用 Pascal 表示法定义，即 `name: type`，参数之间使用逗号分隔，必须显式地为每个参数指定类型。

```kotlin
fun powerOf(number: Int, exponent: Int) {
...
}
```

### 默认参数

函数的参数可以具有默认值，当调用函数时省略了某个参数，则该参数就会使用默认值。与其他语言相比，这样可以减少很多重载方法。

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {
...
}
```

在参数的类型后面添加 `=` 和值来为参数定义默认值。

覆盖方法总与基类方法具有相同的默认参数值，在覆盖带有默认参数值的方法时，必须在签名中省略默认参数值：

```kotlin
open class A {
    open fun foo(i: Int = 10) { ... }
}

class B : A() {
    override fun foo(i: Int) { ... }  // 不允许添加默认值
}
```

### 命名参数

在调用函数时，可以命名函数的参数，这在函数具有大量参数或默认参数的时候非常方便。

比如对于下面的函数：

```kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
...
}
```

可以使用默认参数来调用：

```kotlin
reformat(str)
```

也可以不使用默认参数，就像：

```kotlin
reformat(str, true, true, false, '_')
```

使用命名参数可以提高代码的可读性：

```kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

如果不需要修改所有的参数：

```kotlin
reformat(str, wordSeparator = '_')
```

注意命名参数的语法不能用于调用 Java 方法，因为 Java 的字节码并不总会保留函数的参数名称。

### 返回 Unit 的函数

如果一个函数并不返回任何有用的值，则它的返回类型是 `Unit`。`Unit` 类型只有一个值——`Unit`，这个值不需要被显式地返回：

```kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` or `return` is optional
}
```

`Unit` 返回类型的声明是可选的，下面代码的函数声明和上面等效：

```kotlin
fun printHello(name: String?) {
    ...
}
```

### 单表达式函数

如果一个函数仅返回一个表达式，则可以省略大括号，把函数体直接写在 `=` 后面：

```kotlin
fun double(x: Int): Int = x * 2
```

当编译器可以推断出类型时，返回值类型的声明是[可选的](#显式返回类型):

```kotlin
fun double(x: Int) = x * 2
```

### 显式返回类型

以代码块作为函数体的函数必须始终显式地指定返回值的类型，除非函数返回的是 `Unit`，[此时可以省略返回值类型](#返回-unit-的函数)。Kotlin 不会推断以代码块作为函数体的函数的返回值类型，因为此类函数往往有复杂的控制流，其返回值类型对读者来说并不明显（有时对编译器也是如此）。

### 可变数量的参数（Varargs）

函数的参数（通常是最后一个）可以使用 `vararg` 修饰符标记：

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

`vararg` 允许向函数传递可变数量的参数：

```kotlin
val list = asList(1, 2, 3)
```

在函数内部，一个类型为 `T` 的 `vararg` 参数表现为一个 `T` 型的数组，也就是说，上面代码中 `ts` 的类型为 `Array<out T>`。

只能有一个参数被标记为 `vararg`。如果 `vararg` 参数不是参数列表中的最后一个参数，则 `vararg` 参数后的其他参数可以以命名参数的形式传递，或者如果参数是函数类型，可以以括号外 Lambda 表达式的形式传递。

在调用具有可变参数的函数时，我们可以逐一地传递参数，如 `asList(1, 2, 3)`。如果想把一个已有数组的内容传递给函数，可以使用**传播**（Spread）操作符（数组前缀 `*`）：

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```


## 函数作用域

在 Kotlin 中，可以在文件的顶层声明函数，这意味着不需要像 Java、C#、Scala 等语言一样创建类来持有函数。除了顶层函数，Kotlin 的函数还可以声明为局部函数、作为成员函数或者扩展函数。

### 局部函数

Kotlin 支持局部函数，也就是在函数内的函数。

```kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

局部函数可以访问外部函数的局部变量（即闭包，Closure），所以在上面的例子里，可以把参数 `visited` 声明为局部变量：

```kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

### 成员函数

成员函数是定义在类或 object 中的函数。

```kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

使用点（`.`）表示法来调用成员函数：

```kotlin
Sample().foo() // 创建 Sample 类的实例并调用 foo
```

关于类和覆盖成员的更多内容，请参考[类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/classes-and-inheritance.md#类)和[继承](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/classes-and-inheritance.md#继承)。


## 泛型函数

函数可以有泛型参数，在函数名前面使用尖括号指明泛型参数。

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}
```

关于泛型的更多内容，请参考[泛型](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/generics.md)。


## 内联函数

内联函数的具体解释见[这里](https://blog.nex3z.com/2017/06/11/kotlin-reference-inline-functions/)。


## 扩展函数

扩展函数的具体解释在[这里](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md)。


## 高阶函数和 Lambda

高阶函数和 Lambda 表达式的具体解释见[这里](https://blog.nex3z.com/2017/06/11/kotlin-reference-higher-order-functions-lambdas/)。


## 尾递归函数

Kotlin 支持[尾递归（Tail Recursion）](https://en.wikipedia.org/wiki/Tail_call)的函数式编程风格，可以用递归的形式实现一些通常被写为循环的算法，而不必担心栈溢出。当函数被标记为 `tailrec` 且满足特定要求时，编译器会把递归优化为快速高效的循环。

```kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

上面的函数计算了余弦函数的不动点，是一个数学常量，只是从 1.0 开始反复调用 `Math.cos`，直到结果不再变化，输出的结果为 0.7390851332151607。上面的代码等效于下面这段传统风格的代码：

```kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

要使用尾递归，除了使用 `tailrec` 修饰符，还要求函数在其最后一步操作中调用自身，如果在自身调用之后还有其他代码，就不能使用尾递归。尾递归不能用于 try/catch/finally 块中。目前仅 JVM 平台支持尾递归。
