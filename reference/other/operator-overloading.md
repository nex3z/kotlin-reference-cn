# 操作符重载

Kotlin 允许我们为类型上的一组预定义的操作符提供实现。这些操作符具有固定的符号表现形式（如 `+` 或者 `*`）和固定的[优先级](https://kotlinlang.org/docs/reference/grammar.html#precedence)。可以通过固定名称的[成员函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/functions.md#成员函数)或[扩展函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md#扩展函数)来为对应类型（二元操作符左值的类型，或一元操作符的参数类型）提供操作符的实现。重载操作符的函数需要使用 `operator` 修饰符标记。

下面描述了重载不同操作符的约定。

## 一元操作

### 一元前缀操作符

| 表达式 | 转换为           |
|--------|------------------|
| `+a`   | `a.unaryPlus()`  |
| `-a`   | `a.unaryMinus()` |
| `!a`   | `a.not()`        |

上表说明，编译器处理在如 `+a` 的表达式时，会进行如下的步骤：

- 判断 `a` 的类型，这里记为 `T`。
- 搜索以 `T` 为接收者的，名为 `unaryPlus()` 且带有 `operator` 修饰符的无参函数（成员函数或扩展函数）。
- 如果该函数不存在或不明确，则给出编译错误。
- 如果该函数存在且返回类型是 `R`，则表达式 `+a` 的类型为 `R`。

注意这些操作对[基本类型](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/basics/basic-types.md)有优化，不会引入函数调用的开销。

举例来说，下面是重载一元减号操作符的方法：

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)
println(-point)  // 输出 "(-10, -20)"
```

### 递增和递减

| 表达式 | 转换为           |
|--------|------------------|
| `a++`  | `a.inc()` + 见下 |
| `a--`  | `a.dec()` + 见下 |

`inc()` 和 `dec()` 函数必须有一个返回值，该值会赋给进行了 `++` 或 `--` 操作的变量。它们不应该改变调用 `inc` 或 `dec` 的对象。

编译器通过以下步骤来解析*后缀*形式的操作符，如 `a++`:

- 判断 `a` 的类型，这里记为 `T`。
- 搜索适用于类型为 `T` 的接收者的，名为 `inc()` 且带有 `operator` 修饰符的无参函数。
- 检查返回类型是否为 `T` 的子类。

计算表达式的作用为：

- 把 `a` 的初始值临时存储到 `a0`，
- 把 `a.inc()` 的结果赋给 `a`,
- 返回 `a0` 作为表达式的结果。

`a--` 的相关步骤完全一致。

对于如 `++a` 和 `--a` 的*前缀*形式，解析方式与上面一样，作用为：

- 把 `a.inc()` 的结果赋给 `a`,
- 返回 `a` 的新值作为表达式的结果。


## 二元操作

### 算术运算符

| 表达式  | 转换为                          |
|---------|---------------------------------|
| `a + b` | `a.plus(b)`                     |
| `a - b` | `a.minus(b)`                    |
| `a * b` | `a.times(b)`                    |
| `a / b` | `a.div(b)`                      |
| `a % b` | `a.rem(b)`, `a.mod(b)`（已废弃） |
| `a..b`  | `a.rangeTo(b)`                  |

对于上表中的从操作符，编译器只是把表达式解析成“*转换为*”一栏的内容。

注意 Kotlin 1.1 支持 `ram` 操作符。Kotlin 1.0 使用的 `mod` 操作符在 Kotlin 1.1 中已被废弃。

#### 例子

下面示例的 `Counter` 类以指定数值开始，使用重载的 `+` 操作符增加数值：

```kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}
```

### In 操作符

| 表达式    | 转换为                          |
|-----------|---------------------------------|
| `a in b`  | `b.contains(a)`                 |
| `a !in b` | `!b.contains(a)`                |

对于 `in` 和 `!in` 的处理流程是相同的，但参数的顺序反转了。

### 索引访问操作符

| 表达式                 | 转换为                    |
|------------------------|---------------------------|
| `a[i]`                 | `a.get(i)`                |
| `a[i, j]`              | `a.get(i, j)`             |
| `a[i_1, ..., i_n]`     | `a.get(i_1, ..., i_n)`    |
| `a[i] = b`             | `a.set(i, b)`             |
| `a[i, j] = b`          | `a.set(i, j, b)`          |
| `a[i_1, ..., i_n] = b` | `a.set(i_1, ..., i_n, b)` |

方括号会转换为对 `get` 和 `set` 的调用，并会携带恰当数量的参数。

### 调用操作符

| 表达式                 | 转换为                    |
|------------------------|---------------------------|
| `a()`                  | `a.invoke()`              |
| `a(i)`                 | `a.invoke(i)`             |
| `a(i, j)`              | `a.invoke(i, j)`          |
| `a(i_1, ..., i_n)`     | `a.invoke(i_1, ..., i_n)` |

圆括号会翻译为对 `invoke` 的调用，并会携带恰当数量的参数。

### 增量赋值

| 表达式   | 转换为             |
|----------|--------------------|
| `a += b` | `a.plusAssign(b)`  |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)`   |
| `a %= b` | `a.modAssign(b)`   |

对于赋值操作符，如 `a += b`，编译器会进行如下步骤：

- 如果上表右栏中的函数可用
  - 如果对应的二元函数（如对应 `plusAssign()` 的 `plus()`）也可用，则报告错误（出现混淆）。
  - 确定返回值类型是 `Unit`，否则报告错误。
  - 生成 `a.plusAssign(b)` 的代码。
- 否则，尝试为 `a = a + b` 生成代码（这里会包含类型检查：`a + b` 的类型必须是 `a` 的子类）。

　　注意：在 Kotlin 中，赋值不是表达式。

### 相等和不等操作符

| 表达式                 | 转换为                            |
|------------------------|-----------------------------------|
| `a == b`               | `a?.equals(b) ?: (b === null)`    |
| `a != b`               | `!(a?.equals(b) ?: (b === null))` |

注意：不能重载 `===` 和 `!==`，没有对它们的约定。

`==` 操作的特殊之处在于：它被转换成一个复杂的表达式，用于筛选 `null`。`null == null` 始终为 `true`；对于非 `null` 的 `x`，`x == null` 始终为 `false`，且不会调用 `x.equals()`。

### 比较操作符

| 表达式                 | 转换为                    |
|------------------------|---------------------------|
| `a > b`                | `a.compareTo(b) > 0`      |
| `a < b`                | `a.compareTo(b) < 0`      |
| `a >= b`               | `a.compareTo(b) >= 0`     |
| `a <= b`               | `a.compareTo(b) <= 0`     |

所有的比较都会转换为对 `compareTo` 的调用，该函数要求返回 `Int`。


## 中缀调用命名函数

我们可以通过[中缀函数调用](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/functions.md#中缀表示法)模拟自定义中缀操作。

