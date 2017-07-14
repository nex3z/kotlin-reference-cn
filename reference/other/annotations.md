# 注解

## 注解声明

注解可以为代码添加元数据。在类前使用 `annotation` 修饰符声明注解：

```kotlin
annotation class Fancy
```

在注解类上使用元注解可以为注解添加额外的属性：

- [`@Target`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html) 指定可以被注解的元素类型（类、函数、属性、表达式等）；
- [`@Retention`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html) 指定注解是否被存储到编译后的类文件中，以及是否在运行时能通过反射可见（默认二者都为是）；
- [`@Repeatable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html) 允许在一个元素上多次使用同一个注解；
- [`@MustBeDocumented`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html) 指定注解是公开 API 的一部分，应当被包含在所生成的 API 文档的类或方法的签名中。

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

### 用法

```kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```

如果要注解类的主构造器，就需要为构造器的声明添加 `constructor` 关键字，并在前面添加注解：

```kotlin
class Foo @Inject constructor(dependency: MyDependency) {
    // ...
}
```

也可以注解属性的访问器：

```kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### 构造器

注解可以有带参数的构造器。

```kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

允许使用的参数类型有：

- 对应 Java 基本类型的类型（Int、Long 等）；
- 字符串；
- 类（`Foo::class`）；
- 枚举；
- 其他注解；
- 以上类型的数组。

注解参数不能是可为空的类型（Nullable Type），因为 JVM 不支持在注解属性中保存 `null`。

如果一个注解被用作另一个注解的参数，它的名称不需要使用 `@` 前缀：

```kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

如果要使用类作为注解参数，则需要使用 Kotlin 类（[KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)），Kotlin 编译器会自动把它转换为一个 Java 类，以便 Java 代码能够正常地看到注解和参数。

```kotlin
import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```

### Lambda

注解也可以同于 Lambda，它们会被应用在 `invoke()` 方法上（Lambda 代码体生成在 `invoke()` 里面）。这对于如 [Quasar](http://www.paralleluniverse.co/quasar/) 等框架很有用，Quasar 使用注解来进行并发控制。

```kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```


## 注解使用处目标

<a name="注1返回"></a>
当你注解属性或主构造器参数时，由于对应的 Kotlin 元素会生成多个 Java 元素，因此在生成的 Java 字节码中也存在多个注解位置[【注 1】](#注1)。为了指定注解的具体生成方式，可以使用如下的语法：

```kotlin
class Example(@field:Ann val foo,    // 注解 Java 的字段
              @get:Ann val bar,      // 注解 Java 的 getter
              @param:Ann val quux)   // 注解 Java 的构造器参数

同样的语法也可以用于整个文件，把目标为 `file` 的注解放在文件的顶层、包和导入（如果文件在默认包中的话）之前：

```kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

如果一个目标有多个注解，可以在目标后添加方括号并把所有注解放在方括号内，避免重复指定目标：

```kotlin
class Example {
     @set:[Inject VisibleForTesting]
     var collaborator: Collaborator
}
```

下面列出了支持的使用处目标（Use-Site Target）：

- `file`
- `property`（此目标的注解对 Java 不可见）
- `field`
- `get`（属性的 getter）
- `set`（属性的 setter）
- `receiver`（扩展函数或属性的接收者参数）
- `param`（构造器参数）
- `setparam`（属性 setter 参数）
- `delegate`（为委托属性存储委托实例的字段）

使用如下的语法注解扩展函数的接收者类型：

```kotlin
fun @receiver:Fancy String.myExtension() { }
```

如果不指定使用处目标，目标会根据所使用注解的 `@Target` 注解来选择。如果有多个可用的目标，会使用下面所列的第一个可用目标：

- `param`
- `property`
- `field`

## Java 注解

Kotlin 完全兼容 Java 的注解：

```kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests {
    // apply @Rule annotation to property getter
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```

由于 Java 中编写的注解没有定义参数的顺序，所以不能使用常规的函数调用语法来传递参数，需要使用命名参数语法。

```java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

```kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

就像在 Java 中一样，对于 `value` 参数这一特殊情况，它的值可以不使用显式的名称指定。

```java
// Java
public @interface AnnWithValue {
    String value();
}
```

```kotlin
// Kotlin
@AnnWithValue("abc") class C
```

如果 `value` 参数是数组类型，它在 Kotlin 中会变为 `vararg` 参数：

```java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

```kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

对于其他数组类型的参数，需要显式地使用 `arrayOf`：

```java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```

```kotlin
// Kotlin
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar")) class C
```

注解实例的值以属性的形式暴露给 Kolin 代码：

```java
// Java
public @interface Ann {
    int value();
}
```

```kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```


---
<a name="注1"></a>【注 1】如主构造器中声明的属性同时对应主构造器参数、类的属性及访问器，如果对其进行注解，可以指定注解要用于主构造器参数和（或）类的属性和（或）属性的访问器。[【返回】](#注1返回)
