# 返回和跳转

Kotlin 有三种结构跳转表达式：

- `return`：默认返回最近一层的函数或[匿名函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md#匿名函数)。
- `break`：结束最近一层的循环。
- `continue`：进行到最近一层循环的下一步。

这三个表达式都可以用在其他表达式中，如：

```kotlin
val s = person.name ?: return
```

<a name="注1返回"></a>
以上三个表达式的类型为 [Nothing](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/exceptions.md#nothing-类型)[【注 1】](#注1)。

## break 和 continue 标签

可以为 Kotlin 中的任意表达式添加标签，标签的形式为标识符后加 `@` 符号，如 `abc@`、`fooBar@` 等（详见 [Grammar](https://kotlinlang.org/docs/reference/grammar.html#labelReference)）。只需把标签放在表达式前面，即可为表达式添加标签：

```kotlin
loop@ for (i in 1..100) {
    // ...
}
```

现在，我们就可以通过标签来限定 `break` 和 `continue`：

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

使用标签限定的 `break` 会让程序跳转到标签对应的循环之后，`continue` 则会继续执行该循环的下一步迭代。


## 标签处返回

Kotlin 中的函数可以通过函数字面值、局部函数和对象表达式（Object Expression）等方式进行嵌套，通过标签限定的 `return` 可以直接让外部函数返回。这种用法最重要的用途是从 Lambda 中返回，比如当我们这样写：

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return  // 在 Lambda 中的非局部返回会从 Lambda 直接返回到 foo() 的调用者
        print(it)
    }
}
```

<a name="注2返回"></a>
上面代码中的 `return` 会返回最近一层的函数，也就是 `foo()`。（注意此类非局部的返回仅可用于传递给[内联函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/inline-functions.md)的 Lambda 表达式。[【注 2】](#注2)）如果想要让 Lambda 表达式返回，则必须为其添加标签，并使用标签限定的 `return`：

```kotlin
fun foo() {
    ints.forEach lit@ {
        if (it == 0) return@lit
        print(it)
    }
}
```

现在，上面代码中的 `return` 就只会让 Lambda 表达式返回。通常使用隐式的标签会更方便：隐式标签和接受 Lambda 表达式的函数同名：

```kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

此外，还可以使用[匿名函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md#匿名函数)代替 Lambda 表达式，匿名函数中的 `return` 会让匿名函数本身返回：

```kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // 局部返回到匿名函数的调用者，也就是 forEach 循环
        print(value)
    })
}
```

<a name="注3返回"></a>
当标签和返回值同时出现在 `return` 后面时，分词器会优先标签限定的返回[【注 3】](#注3)，如：

```kotlin
return@a 1
```

表示“在 `@a` 标签处返回 `1`”，而不是“返回一个带有标签的表达式 `(@a 1)`”。


---
<a name="注1"></a>【注 1】Nothing 类型是一个特殊的类型，它没有实际的值，只是用来标记返回或不可能执行到的位置（如抛出异常）。[【返回】](#注1返回)

<a name="注2"></a>【注 2】在 Kotlin 中，未限定的 `return` 只能用于退出一个命名或匿名函数；如果想要退出 Lambda，就必须使用标签。但如果 Lambda 传递到的函数是内联的，则 Lambda 中的 `return` 也会被内联，所以此时允许使用 `return`。[【返回】](#注2返回)

<a name="注3"></a>【注 3】即同时有标签和返回值时， `return` 会优先结合标签，构成带标签限定的返回。[【返回】](#注3返回)
