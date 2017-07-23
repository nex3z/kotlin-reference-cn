# 基本类型

Kotlin 中一切皆对象，也就是说，我们可以调用任意变量的成员函数和属性。部分类型具有特殊的内部表示方法，如数字、字符和布尔类型在运行时可以用原始值（Primitive Value）表示，但从用户的角度来看，它们仍表现为普通的类。本节描述了 Kotlin 中的这些基本类型：数字、字符、布尔、数组和字符串。


## 数字

Kotlin 对数字的处理和 Java 类似，但又不完全相同。比如说，Kotlin 不会对数字进行隐式的扩大转换，字面值的表现也有不同。

Kotlin 提供了以下表示数字的内置类型（这一点和 Java 类似）：

| 类型   | 位宽 |
|--------|------|
| Double | 64   |
| Float  | 32   |
| Long   | 64   |
| Int    | 32   |
| Short  | 16   |
| Byte   | 8    |

注意 Kotlin 中的字符不是数字。

### 字面常量

以下方法都可以用来表示整形字面常量：

- 十进制：`123`
  - 长整形以大写的 `L` 结尾：`123L`
- 十六进制：`0x0F`
- 二进制：`0b00001011`

注意：Kotlin 不支持八进制字面值。

Kotlin 也支持传统的浮点数字表示方法：

- 默认为双精度：`123.5`，`123.5e10`
- 单精度浮点以 `f` 或 `F` 结尾：`123.5f`

### 在数值字面值中使用下划线 (起自 1.1)

可以在数字字面值中插入下划线来提高可读性，如：

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 表示方法

在 Java 平台上，数值类型以 JVM 基本类型（Primitive Type）的形式存储。当需要可为空（Nullable）的数值引用（如 Int?）或泛型时，数值类型会被装箱（Boxing）。

<a name="注1返回"></a>
注意装箱后的数值不一定会保持一致性（Identity）：

```kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

但会保持相等性（Equality）：

```kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```
[【注 1】](#注1)

### 显式转换

由于采用了不同的表示方法，较小的类型不是较大类型的子类，否则就会有下面的问题：

```kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(a == b) // Surprise! This prints "false" as Long's equals() check for other part to be Long as well
```

由此导致的结果是，相等性也悄悄地被破坏了。

因此，较小的类型不会隐式地转换为较大的类型。这意味着，我们不能直接把一个 `Byte` 类型的值赋给一个 `Int` 类型的变量：

```kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```

但可以通过显式地转换来扩展数值的范围：

```kotlin
val i: Int = b.toInt() // OK: explicitly widened
```

所有数值类型都提供了如下的转换方法：

- `toByte(): Byte`
- `toShort(): Short`
- `toInt(): Int`
- `toLong(): Long`
- `toFloat(): Float`
- `toDouble(): Double`
- `toChar(): Char`

<a name="注2返回"></a>
缺乏隐式转换并不会带来明显的问题，因为实际类型可以通过上下文推断出来，且操作符已经为合适的转换进行了重载，例如：

```kotlin
val l = 1L + 3 // Long + Int => Long
```

[【注 2】](#注2)

### 运算操作

Kotlin 支持数值类型间的一系列标准运算，这些运算都声明为了对应类的方法（编译器会把调用优化为对应的指令），见 [操作符重载](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/operator-overloading.md)。

对于位运算，并没有专用的操作符，但可以用中缀的形式调用命名函数，如：

```kotlin
val x = (1 shl 2) and 0x000FF000
```

完整的位运算操作符列举如下（仅可用于 `Int` 和 `Long`）：

- `shl(bits)` – 带符号左移（Java 中的 `<<`）
- `shr(bits)` – 带符号右移（Java 中的 `>>`）
- `ushr(bits)` – 无符号左移（Java 中的 `>>>`）
- `and(bits)` – 按位与
- `or(bits)` – 按位或
- `xor(bits)` – 按位异或
- `inv()` – 按位反转


## 字符

`Char` 类型用于表示字符，不能把它直接当成数字使用：

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

字符字面值使用单引号：`'1'`。可以使用反斜杠来转义特殊字符，支持的转义序列有：`\t`， `\b`， `\n`， `\r`， `\'`， `\"`， `\\` 和 `\$`。如果要编码其他字符，可以使用 Unicode 转义序列的语法：`'\uFF00'`。

我们可以显式地把字符转换为 `Int` 型数值：

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

当需要可为空的引用时，字符类型也会被装箱，就像数值类型一样。字符类型装箱后也不保持一致性。


## 布尔

`Boolean` 表示布尔类型，有 true 和 false 两个值。`Boolean` 也会在需要可为空的引用时装箱，内置的操作有：

- `||`：短路或
- `&&`：短路与
- `!`：取反


## 数组

`Array` 表示数组类型，具有 `get` 和 `set` 方法（通过操作符重载为 `[]`）和 `size` 属性，以及其他一些常用成员函数：

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

可以使用库函数 `arrayOf()` 并传递初始值来创建数组，如 `arrayOf(1, 2, 3)` 会创建数组 [1, 2, 3]。此外，使用库函数 `arrayOfNulls()` 可以创建指定大小、以 null 填充的数组。

还可以使用工厂函数创建数组，需要提供数组容量和一个计算数组各个位置上的值的函数：

```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

就像上面提到的，`[]` 操作表示调用 `get()` 和 `set()` 成员函数。

注意：Kotlin 中的数组是不变量，这一点和 Java 不同。这意味着 Kotlin 不允许把 `Array<String>` 赋给 `Array<Any>`，因为这样可能会带来运行时错误（但可以使用 `Array<out Any>`，见[类型投射](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/generics.md#类型投射)）。

Kotlin 提供了一系列专用的类来表示基本类型数组，以避免装箱带来的开销，如 `ByteArray`、 `ShortArray`、 `IntArray` 等，这些数组并不继承自 `Array` 类，但提供了和 `Array` 相同的方法和属性。它们也有对应的工厂方法，如：

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```


## 字符串

`String` 表示字符串类型。，String 是不可变的，其中的各个字符可以使用索引操作来访问，如 `s[i]`。可以使用 for 循环遍历 String：

```kotlin
for (c in str) {
    println(c)
}
```

### 字符串字面值

Kotlin 有两种字符串字面值表示方法，带转义的字符串（Escaped String）可以包含转义字符，原始字符串（Raw String）可以包含换行（Newline）和任意文本。带转义的字符串（Escaped String）和 Java 的字符串非常类似：

```kotlin
val s = "Hello, world!\n"
```

原始字符串使用三个双引号包围，不需要转义，可以包含换行和任意字符，如：

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

可以使用 [`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) 函数移除开头的空格：

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

这里的 `|` 是默认的边界标识符，也可以使用其他字符，只需以参数的形式传递给 `trimMargin()` 即可，如 `trimMargin(">")`。

### 字符串模板

字符串可以包含模板表达式（Template Expression），表达式的值会被计算并拼接在字符串中。模板表达式以 `$` 符号开头，可以包含简单的名称：

```kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

或者包围在大括号里的任意表达式：

```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

原始字符串和带转义的字符串都支持模板。如果要在原始字符串中加入 `$` 字符（原始字符串不支持反斜杠转义），需要使用如下的语法：
```kotlin
val price = """
${'$'}9.99
"""
```


---

<a name="注1"></a>【注 1】Kotlin 中，`===` 检查的是引用的相等性，即检查两个引用是否指向同一个对象；`==` 检查的是结构的相等性，即检查 `equals()`。更多信息可参考[相等性](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/equality.md)。[【返回】](#注1返回)

<a name="注2"></a>【注 2】在 `val l = 1L + 3 // Long + Int => Long` 中，等号右边 `1L` 是 `Long` 型，`3` 是 `Int` 型。`Long` 的 `+` 操作符有针对 `Int` 的[重载](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-long/plus.html)：

```kotlin
operator fun plus(other: Int): Long (source)
```

即 `Long + Int` 的结果是 `Long`。故等号左边 `val l` 的类型是 `Long`。操作符重载的更多内容见[操作符重载](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/operator-overloading.md)。[【返回】](#注2返回)
