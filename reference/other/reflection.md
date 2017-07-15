# 反射

反射是用于在运行时内省程序内部结构的一组语言和库的功能。Kotlin 的函数和属性是语言的一等公民，对其的内省（在运行时获取属性或函数的名称或类型）与函数式或响应式的风格密切相关。

在 Java 平台上，反射功能所需的运行时组件是以单独的 JAR 文件的形式发布的（`kotlin-reflect.jar`），这是为了减小不使用反射功能的应用的运行库尺寸。如果你要使用反射，请确保把该 .jar 文件加入到工程的 classpath 里。


## 类引用

获取 Kotlin 类的运行时引用是最基础的反射功能。使用*类字面值*（Class Literal）语法获取静态已知的 Kotlin 类的引用。

```kotlin
val c = MyClass::class
```

该引用值的类型是 [KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)。

注意 Kotlin 类的引用与 Java 类的引用不同。使用 `KClass` 实例的 `.java` 属性获取 Java 类的引用。


## 绑定类引用（起自 1.1）

同样的 `::class` 语法如果以某个对象作为接收者来使用，就可以获得该对象的类的引用：

```kotlin
val widget: Widget = ...
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

无论接收者表达式是什么类型（`Widget`），你得到是该对象实际的类的引用，比如 `GoodWidget` 或 `BadWidget`。


## 函数引用

声明如下的命名函数：

```kotlin
fun isOdd(x: Int) = x % 2 != 0
```

我们可以简单地直接调用该函数（`isOdd(5)`），也可以使用 `::` 操作符，以值的形式传递它，如传递给另一个函数：

```kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 输出 [1, 3]
```

这里 `::isOdd` 是一个函数类型 `(Int) -> Boolean` 的值。

当期望的类型对上下文已知时，`::` 也可以用于重载函数，如：

```kotlin
fun isOdd(x: Int) = x % 2 != 0
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 引用 isOdd(x: Int)
```

另外，也可以通过把方法引用存储到有显式类型的变量，来提供所需的上下文：

```kotlin
val predicate: (String) -> Boolean = ::isOdd  // 引用 isOdd(x: String)
```

如果需要使用类的成员或扩展函数，就需要使用限定。如 `String::toCharArray` 表示的是一个 `String` 的类型为 `String.() -> CharArray` 的扩展函数。

### 例: 函数组合

考虑下面的函数：

```kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

它返回的是传递给它的两个函数的组合：`compose(f, g) = f(g(*))`，可以把它用在可调用（Callable）的引用上：

```kotlin
fun length(s: String) = s.length

val oddLength = compose(::isOdd, ::length)
val strings = listOf("a", "ab", "abc")

println(strings.filter(oddLength)) // 输出 "[a, abc]"
```


## 属性引用

使用 `::` 操作符以一等对象（First-Class Object）的形式访问属性：

```kotlin
var x = 1

fun main(args: Array<String>) {
    println(::x.get()) // 输出 "1"
    ::x.set(2)
    println(x)         // 输出 "2"
}
```

表达式 `::x` 表示类型为 `KProperty<Int>` 的属性对象，该对象允许我们使用 `get()` 读取它的值，或者使用 `name` 属性获取它的名称。更多信息请参考[关于 `KProperty` 类的文档](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html)。

对于可变属性，如 `var y = 1`，`::y` 返回的是 [`KMutableProperty<Int>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html) 类型的值，它有一个 `set()` 方法。

属性引用可以用于无参函数的场景：

```kotlin
val strs = listOf("a", "bc", "def")
println(strs.map(String::length)) // 输出 [1, 2, 3]
```

访问作为类成员的属性，可以使用如下的限定形式：

```kotlin
class A(val p: Int)

fun main(args: Array<String>) {
    val prop = A::p
    println(prop.get(A(1))) // 输出 "1"
}
```

对于扩展属性：

```kotlin
val String.lastChar: Char
    get() = this[length - 1]

fun main(args: Array<String>) {
    println(String::lastChar.get("abc")) // 输出 "c"
}
```

### 与 Java 反射的互可操作性

在 Java 平台，标准库包含了反射类的扩展，提供对 Java 反射对象的双向映射（见 `kotlin.reflect.jvm` 包）。举例来说，可以使用下面的方法查找一个支持字段，或者一个用作 getter 的 Java 方法：

```kotlin
import kotlin.reflect.jvm.*
 
class A(val p: Int)
 
fun main(args: Array<String>) {
    println(A::p.javaGetter) // 输出 "public final int A.getP()"
    println(A::p.javaField)  // 输出 "private final int A.p"
}
```

使用 `.kotlin` 扩展属性获取对应 Java 类的 Kotlin 类：

```kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```


## 构造器引用

可以像方法和属性一样引用构造器，构造器引用可以用于期望函数类型对象、使用与构造器相同的参数、且返回合适类型的对象的位置。使用 `::` 操作符加类名来引用构造器。

下面的函数期望一个无参的、返回类型为 `Foo` 函数参数：

```kotlin
class Foo

fun function(factory: () -> Foo) {
    val x: Foo = factory()
}
```

使用 `::Foo`，即类 `Foo` 的无参构造器，可以像下面一样简单地进行调用：

```kotlin
function(::Foo)
```


## 绑定函数和属性引用（起自 1.1）

你可以引用一个特定对象的实例方法。

```kotlin
val numberRegex = "\\d+".toRegex()
println(numberRegex.matches("29")) // prints "true"
 
val isNumber = numberRegex::matches
println(isNumber("29")) // prints "true"
```

我们存储了方法 `matches` 的引用，而不是直接调用该方法。此类引用是与其接收者绑定的，可以直接调用（就像上面的例子），或者在以函数表达式的形式使用：

```kotlin
val strings = listOf("abc", "124", "a70")
println(strings.filter(numberRegex::matches)) // prints "[124]"
```

绑定和非绑定引用相比，绑定的可调用引用附带其接收者，所以不再需要接收者类型作为参数：

```kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches

val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```

属性引用也可以是绑定的：

```kotlin
val prop = "abc"::length
println(prop.get())   // 输出 "3"
```
