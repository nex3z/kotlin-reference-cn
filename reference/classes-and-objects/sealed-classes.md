# 密封类

密封类（Sealed Class）用于表示受限制的类层级结构，比如约束一个值的类型只能是某个有限类型集合中的类型之一，而不能是其他类型。某种意义上，这类似于枚举的扩展：枚举类型可以取的值也是受限制的，但每个枚举常量仅作为单个实例存在，而密封类的子类可以有多个实例，且可以具有状态。

通过在类名前添加 `sealed` 关键字来声明密封类，密封类可以有子类，但所有的子类必须与父密封类位于同一个文件中（在 Kotlin 1.1 版本前，这一约束更加严格，密封类的所有子类必须嵌套在密封类声明的内部）。

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

（上面的例子用到了 Kotlin 1.1 的一个新功能：数据类可以继承其他类，包括密封类。）

继承密封类子类的类（密封类的间接子类）可以定义在任何地方，不需要在同一个文件中。

密封类的主要价值在于结合 [when 表达式](https://blog.nex3z.com/2017/06/01/kotlin-reference-control-flow/#When_Expression) 使用。如果可以证明语句已经覆盖了所有的情况，则不必为语句添加 `else` 子句。

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 因为我们已经覆盖了所有的情况，故不再需要 else 子句
}
```
