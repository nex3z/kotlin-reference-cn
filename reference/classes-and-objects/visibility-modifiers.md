# 可见性修饰符

类、`object`、接口、构造器、函数、属性及其 setter 可以具有可见性修饰符（属性的 getter 始终与属性本身具有同样的可见性）。Kotlin 中有四种可见性修饰符：`private`、`protected`、`internal` 和 `public`，如果没有使用显式的修饰符，则默认的可见性为 `public`。


## 包

函数、属性、类、`object` 和接口可以声明在顶层，也就是直接在包内：

```kotlin
// file name: example.kt
package foo

fun baz() {}
class Bar {}
```

- 如果没有指定可见性修饰符，则默认为 `public`，这意味着该声明在任何地方都可见。
- 如果把声明标记为 `private`，则只在该声明所在的文件内可见。
- 如果把声明标记为 `internal`，则在同一模块（[Module](https://blog.nex3z.com/2017/06/06/kotlin-reference-visibility-modifiers/#Modules)）下可见。
- `protected` 不能用于顶层声明。

例子：
```kotlin
// file name: example.kt
package foo

private fun foo() {} // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt
    
internal val baz = 6    // visible inside the same module
```


## 类和接口

对于在类中声明的成员：

- `private` 表示仅在该类内部可见（包括类的成员）。
- `protected` —— 和 `private` 相同，外加对子类可见。
- `internal` —— 在同一模块下的客户端代码如果能够看到某个类的声明，则也能看到该类中的 `internal` 成员。
- `public` —— 客户端代码如果能看到某个类的声明的，则也能看到该类中的 `public` 成员。

Java 使用者需要注意：在 Kotlin 中，外部类看不到内部类的私有（`private`）成员。

如果覆盖了 `protected` 成员，而没有指定可见性修饰符，则该成员仍具有 `protected` 可见性。

例子：

```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default
    
    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible

    override val b = 5   // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either 
}
```

### 构造器

使用如下语法来指定主构造器的可见性（注意此时必须显式地使用 `constructor` 关键字）：

```kotlin
class C private constructor(a: Int) { ... }
```

这里的构造器是私有的。所有的构造器默认都是 `public` 的，相当于在类可见的地方构造器都可见（亦即 `internal` 的构造器仅在模块内可见）。

### 局部声明

局部变量、方法和类不能具有可见性修饰符。


## 模块

使用 `internal` 修饰的成员仅在模块内可见，具体来说，模块是一组在一起编译的 Kotlin 文件，如：

- 一个 IntelliJ IDEA 模块；
- 一个 Maven 或者 Gradle 项目；
- 一个 Gradle 源码集合（Source Set）；
- 在一次 Ant 任务调用中一起编译的一组文件。
 