# 类型检查和转换

## is 和 !is 操作符

使用 `is`（和其否定形式 `!is`）可以在运行时检查某个对象是否符合给定的类型：

```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // 同 !(obj is String)
    print("Not a String")
}
else {
    print(obj.length)
}
```


## 智能转换

在 Kotlin 中，很多情况下并不需要使用显式的转换操作符，因为编译器会记录对不可变（Immutable）值的 `is` 检查结果，并在需要时自动插入（安全的）转换：

```kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x 会自动转换为 String
    }
}
```

如果否定的检查会导致返回，编译器也聪明到可以判断转换是否安全：

```kotlin
    if (x !is String) return
    print(x.length) // x 会自动转换为 String
```

又或者转换发生在 `&&` 或 `||` 的右边：

```kotlin
    // 在 || 右边，x 会自动转换为 String 
    if (x !is String || x.length == 0) return

    // 在 && 右边，x 会自动转换为 String 
    if (x is String && x.length > 0) {
        print(x.length) // x 会自动转换为 String
    }
```

智能转换也适用于 [`when` 表达式](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/basics/control-flow.md#when-表达式)和 [`while` 循环](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/basics/control-flow.md#while-循环)：

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

如果编译器不能保证变量在检查处和使用处之间保持不变，则不会发生智能转换。具体来说，智能转换是否适用的规则如下：

- `val` 局部变量：始终适用
- `val` 属性：如果属性是 private 或 internal 的，或者在定义属性的同模块内进行了检查，则适用智能转换。智能转换不适用于 open 或有自定义 getter 的属性。
- `var` 局部变量：如果属性在检查和使用之间没有被修改，且没有被 Lambda 捕获和修改，则适用。
- `var` 属性：始终不适用（因为其他代码可以在任意时刻修改该变量）


## “不安全的”转换操作符

通常来说，转换操作会在转换失时抛出异常，因此我们称之为是*不安全的*（Unsafe）。Kotlin 中使用中缀操作符 `as` 进行不安全的转换，（详见 [Operator Precedence](https://kotlinlang.org/docs/reference/grammar.html#precedence)）。

```kotlin
val x: String = y as String
```

注意 `null` 不能被转换为 `String`，因为这个类型不是[可为空的（Nullable）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/null-safety.md) 的。如果 `y` 为 `null` ，则上面的代码会抛出异常。为了达到 Java 中的转换语意，需要在转换的右边使用可为空的类型，像：

```kotlin
val x: String? = y as String?
```


## “安全的”（可为空的）转换操作符

为了避免抛出异常，可以使用*安全*的转换操作符 `as?`，它会在转换失败时返回 `null`：

```kotlin
val x: String? = y as? String
```

注意尽管 `as?` 的右边的 `String` 是不可为 null 的类型，但转换的结果是可为空的。
