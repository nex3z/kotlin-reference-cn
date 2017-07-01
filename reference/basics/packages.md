# 包

可以在源文件的开头进行包声明：

```kotlin
package foo.bar

fun baz() {}

class Goo {}

// ...
```

源文件中的全部内容（如类和函数）都包含在所声明的包中。在上面的例子中，函数 `baz()` 的全名为 `foo.bar.baz`，类 `Goo` 的全名为 `foo.bar.Goo`。

如果没有指定所属的包，则该文件的内容属于没有名称的“默认”包。


## 默认导入

　　下面所列的包会默认导入到每个 Kotlin 文件中:

- kotlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.* (since 1.1)
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*

　　对于具体的目标平台，还会导入额外的包：

- JVM:
  - java.lang.*
  - kotlin.jvm.*
- JS:
  - kotlin.js.*


## 导入

除了默认导入的包，每个文件也可以包含自己的导入指令。[Grammar](https://kotlinlang.org/docs/reference/grammar.html#import) 中描述了导入所使用的语法。

可以导入单个名称：

```kotlin
import foo.Bar // Bar is now accessible without qualification
```

也可以导入整个作用域（包、类、object 等）内所有可以访问的内容：

```kotlin
import foo.* // everything in 'foo' becomes accessible
```

如果存在命名冲突，可以使用 `as` 关键字在本地为冲突的对象进行重命名，以消除歧义：

```kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```

`import` 关键字并不仅限于导入类，还可以用它来导入其他声明，如：

- 顶层函数和属性；
- 在[对象声明](https://blog.nex3z.com/2017/06/10/kotlin-reference-object-expressions-declarations/#Object_declarations)中的函数和属性；
- [枚举常量](https://blog.nex3z.com/2017/06/09/kotlin-reference-enum-classes/)

Kotlin 中的所有声明的导入都通过常规的 import 关键字进行，没有单独的静态导入（[import static](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html)）语法，这一点与 Java 不同。


## 顶层声明的可见性

如果把一个顶层声明标记为 `private`，则该声明为文件私有（见[可见性修饰符](https://blog.nex3z.com/2017/06/06/kotlin-reference-visibility-modifiers/)）。
