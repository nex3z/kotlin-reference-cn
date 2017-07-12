# 异常

## 异常类

在 Kotlin 中，所有的异常类都是 `Throwable` 的子类。每一个异常都有一个消息、调用栈和一个可选的原因。

使用 `throw` 表达式抛出异常对象：

```kotlin
throw MyException("Hi There!")
```

使用 `try` 表达式捕获异常：

```kotlin
try {
    // 一些代码
}
catch (e: SomeException) {
    // 处理
}
finally {
    // 可选的 finally 块
}
```

可以有零个或多个 `catch` 代码块，`finally` 块可以省略，但必须至少有一个 `catch` 或 `finally`。

### Try 是一个表达式

`try` 是一个表达式，也就是说，它可以有返回值。

```kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException) { null }
```

`try` 表达式的返回值是 `try` 块中的最后一个表达式，或者是 `catch` 块中的最后一个表达式。`finally` 块中的内容不会影响表达式的结果。


## 受检异常

Kotlin 中没有受检异常（Checked Exception），这样做有很多原因，我们会提供一个简单的例子。

下面是 JDK 中的一个接口，由 `StringBuilder` 类实现：

```kotlin
Appendable append(CharSequence csq) throws IOException;
```

该接口的签名表示，每当追加一个字符串到某处（如 `StringBuilder`，某种日志，控制台等），就必须捕获 `IOExceptions`，因为可能在进行 IO 操作（`Writer` 也实现了 `Appendable`）······因此，下面的代码随处可见：

```kotlin
try {
    log.append(message)
}
catch (IOException e) {
    // 一定是安全的
}
```

正如 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) “条目 64：不要忽视异常” 所述，这样并不好。

Bruce Eckel 在 [Does Java need Checked Exceptions?](http://www.mindview.net/Etc/Discussions/CheckedExceptions) 中说：

> 通过对小型程序的审查可以得出这样的结论：规范的异常可以提高开发者的效率，并提高代码质量。但凭借大型软件项目的开发经验则会得出不同的结论：这样做会降低效率，对代码质量的提升也微乎其微。

其他的例证还有：

- [Java's checked exceptions were a mistake](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (Rod Waldhoff)
- [The Trouble with Checked Exceptions](http://www.artima.com/intv/handcuffs.html) (Anders Hejlsberg)


## Nothing 类型

`throw` 在 Kotlin 中也是一个表达式，可以把它作为 Elvis 表达式的一部分：

```kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```

`throw` 表达式的类型是一个特殊的类型 `Nothing`，这个类型没有值，用于标记不可能执行到的代码位置。在代码中，可以使用 `Nothing` 标记从不返回的函数：

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

当你调用这个函数时，编译器知道程序不会继续往下执行。

```kotlin
val s = person.name ?: fail("Name required")
println(s)     // 已知 s 在此处已经初始化了
```


## Java 互可操作性

对于与 Java 的互操作性的信息，请参考 [Java Interoperability](https://kotlinlang.org/docs/reference/java-interop.html) 中关于异常的一节。
