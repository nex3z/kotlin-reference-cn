# 类和继承

## 类

Kotlin 中使用关键字 `class` 声明类：

```kotlin
class Invoice {
}
```

类的声明包括类名、类头（Class Header，用于指定类型参数、主构造器等）和用花括号包围的类体（Class Body）。类头和类体都是可选的，如果没有类体，则花括号也可以省略。

```kotlin
class Empty
```

### 构造器

Kotlin 的类可以有一个**主构造器（Primary Constructor）**，以及一个或多个**次构造器（Secondary Constructor）**。主构造器是类头的一部分，写在类名（和可选的类型参数）之后：

```kotlin
class Person constructor(firstName: String) {
}
```

如果主构造器没有任何可见性修饰符和注解，则可以省略 `constructor` 关键字：

```kotlin
class Person(firstName: String) {
}
```

主构造器中不能包含代码，初始化代码要放在**初始化块（Initializer Blocks）**中，初始化块使用 `init` 关键字作为前缀：

```kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

注意主构造器的参数不仅可以在初始化块中使用，还可以用于初始化在类体中声明的属性：

```kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

实际上，对于声明属性并通过主构造器进行初始化的场景，Kotlin 提供了更简洁的写法：

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ...
}
```

在主构造器中声明的属性可以是可变的（Mutable）`var`，也可以是只读的（Read-Only）`val`，就像普通的属性一样。

如果主构造器有注解或可见性修饰符，则不能省略 `constructor` 关键字：

```kotlin
class Customer public @Inject constructor(name: String) { ... }
```

更多信息请参考[可见性修饰符](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/visibility-modifiers.md#构造器)。

#### 次构造器

在类中还可以声明**次构造器（Secondary Constructor）**，以 `constructor` 关键字作为前缀：

```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

如果一个类已经有了主构造器，那么次构造器都要直接或间接（通过其他次构造器）地委托主构造器，使用 `this` 关键字来委托同一个类中的其他构造器：

```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

如果一个非抽象类没有声明任何构造器（主构造器或次构造器），则它会具有一个自动生成的无参主构造器，该构造器的可见性为 public。如果不想让类具有 public 构造器，则必须手动声明一个具有非默认可见性的空的主构造器：

```kotlin
class DontCreateMe private constructor () {
}
```

**注意**：在 JVM 上，如果主构造器的所有参数都有默认值，则编译器会为其额外生成一个使用了这些默认值的无参构造器，以便于在 Kotlin 中使用如 Jackson 和 JPA 等利用无参构造器创建类的实例的库。

```kotlin
class Customer(val customerName: String = "")
```

### 创建类的实例

直接调用构造器来创建类的实例，就像调用普通的函数一样：

```kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

注意在 Kotlin 中没有 `new` 关键字，

创建嵌套、内部和匿名内部类的方法见 [嵌套类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/nested-classes.md)。

### 类的成员

　　类中可以包含：

- 构造器和初始化块
- [函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/functions.md)
- [属性](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/properties-and-fields.md)
- [嵌套和内部类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/nested-classes.md)
- [对象声明](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/objects.md#对象声明)


## 继承

Kotlin 中的所有类都有一个公共的超类 `Any`，如果一个类没有声明超类，则其默认超类是 `Any`：

```kotlin
class Example // Implicitly inherits from Any
```

<a name="注1返回"></a>
`Any` 与 Java 中的 `java.lang.Object` 不同，它除了 `equals()`、`hashCode()`、和 `toString()` 外没有任何方法。详情请参考 [Java 互操作性](https://kotlinlang.org/docs/reference/java-interop.html#object-methods)。[【注 1】](#注1)

在类头中使用冒号后加类名的形式显式地声明超类：

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

如果子类有主构造器，则必须在主构造器中使用主构造器的参数完成基类的初始化。

如果子类没有主构造器，则其所有的次构造器必须使用 `super` 关键字初始化其基类，或者委托给其他构造器。注意此时不同的次构造器可以调用基类的不同的构造器。

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

类上的 `open` 标注与 Java 中的 `final` 相反，`open` 表示一个类可以被继承，Kotlin 中的类默认都是 `final` 的，对应 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 第 17 条：要么为继承而设计，并提供文档说明，要么就禁止继承（Item 17: Design and document for inheritance or else prohibit it）。

### 覆盖方法

正如上面提到的，我们坚持让 Kotlin 中的事物保持显式。不同于 Java，Kotlin 要求显式地标注可供覆盖的成员（我们称之为 `open`）和进行覆盖的成员：

```kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}
class Derived() : Base() {
    override fun v() {}
}
```

必须使用 `override` 标注 `Derived.v()`，否则编译器就会报错。如果函数没有标注为 `open` ，如 `Base.nv()`，则无论是否使用 `override`，在子类中声明具有同样签名的方法都是非法的。`final` 类（即没有标注为 `open` 的类）不能具有 `open` 成员。

使用 `override` 标注的成员默认是 `open` 的，可以继续被子类覆盖。如果想要禁止再次覆盖，可以使用 `final`：

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

### 覆盖属性

覆盖属性和覆盖方法类似，在派生类中重新定义超类的属性时，必须在前面加上 `override`，且它们必须具有相兼容的类型。已经声明的属性可以被带初始化器的属性或带 getter 方法的属性覆盖：

```kotlin
open class Foo {
    open val x: Int get { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```

可以把 `val` 属性覆盖为 `var`，但不能反过来。因为 `val` 属性本质上声明了一个 getter 方法，而把它覆盖为 `var` 时会在派生类中额外声明一个 setter 方法。

注意可以在主构造器的属性声明中使用 `override` 关键字：

```kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

### 覆盖规则

在 Kotlin 中，实现继承遵守如下的规则：如果一个类从其直接超类中继承了一个成员的多个实现，则它必须要覆盖该成员，并提供自己的实现（可以选择调用继承来的实现之一）。使用 `super` 关键字并在尖括号中限定超类的名称，如 `super<Base>`，来指明所要引用的超类：

```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // 接口的成员默认是 open 的
    fun b() { print("b") }
}

class C() : A(), B {
    // 编译器要求覆盖 f():
    override fun f() {
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
```

`C` 同时继承了 `A` 和 `B`，并继承了 `a()` 和 `b()` 的一个实现，至此没有问题。但 `C` 继承了 `f()` 的两种实现， 所以我们必须在 `C` 中覆盖 `f()` 并给出自己的实现，以消灭歧义。


## 抽象类

类和其中的成员可以声明为 `abstract`，抽象成员在其所在的类中没有实现。注意我们不需要再标记抽象类或函数为 `open`，因为这是理所当然的。

可以把非抽象的 `open` 成员覆盖为抽象成员：

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```


## 伴生对象

Kotlin 的类没有静态方法，这一点与 Java 和 C# 不同。多数情况下，建议使用包级函数来替代。

如果需要让函数能够不依赖对象实例，并能够访问类的内部（如工厂方法），可以把它写为类中[对象声明（Object Declaration ）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/objects.md#对象声明)的成员。

具体来说，如果你在类中声明了一个[伴生对象（Companion Object）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/objects.md#伴生对象)，就可以仅用类名作为限定来调用伴生对象的成员，就像调用 Java / C# 的静态方法一样。


---
<a name="注1"></a>【注 1】当 Java 类型被导入到 Kotlin 时，`java.lang.Object` 的引用会转换为 `Any`，Kotlin 通过[扩展函数（Extension Function）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md#扩展函数)来实现 `java.lang.Object` 的其他方法，如 `wait()`、`notify()`、`getClass()` 等。[【返回】](#注1返回)
