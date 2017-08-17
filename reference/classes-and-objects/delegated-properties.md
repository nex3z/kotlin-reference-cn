# 委托属性

对于一些通用的属性，比起每次都在需要的时候进行手动实现，更好的方法是实现一次并把它放进库里，此类属性如：

- 延迟（Lazy）属性：仅在第一次访问时进行计算，
- 可观察（Observable）属性：属性发生变动时通知监听者，
- 在映射（Map）中存储属性，而不是把属性存储到单独的字段。

Kotlin 提供了委托属性（Delegated Property）来满足此种（及其他）场景：

```kotlin
class Example {
    var p: String by Delegate()
}
```

委托属性的语法为 `val/var <property name>: <Type> by <expression>`，`by` 后面的表达式是*委托*（Delegate），对应属性（`var p`）的 `get()`（和 `set()`）委托给了 `Delegate` 的 `getValue()`（和 `setValue()`）方法。属性委托不需要实现特定接口，但必需要提供 `getValue()`（对于 `var` 属性还需要 `setValue()`）方法，如：

```kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name} in $thisRef.'")
    }
}
```

当我们读取委托给了 `Delegate` 的 `p` 时，`Delegate` 的 `getValue()` 会被调用，`getValue()` 的第一个参数是 `p` 来源的对象，第二个参数是 `p` 本身的描述（如可以从中获得属性的名称），例如：

```kotlin
val e = Example()
println(e.p)
```

输出为：

```
Example@33a17727, thank you for delegating ‘p’ to me!
```

类似的，当我们给 `p` 赋值时，`setValue()` 会被调用，`setValue()` 的前两个参数与 `getValue()` 一样，第三个参数是要赋的值：

```kotlin
e.p = "NEW"
```

上面例子的输出为：

```
NEW has been assigned to ‘p’ in Example@33a17727.
```

委托对象的具体规范和要求见[下面](#属性委托的要求)。

注意从 Kotlin 1.1 起，委托对象可以声明在函数和代码块中，而不必作为类的成员，如[下面的例子](#局部委托属性)。


## 标准委托

Kotlin 标准库提供了多种常用委托的工厂方法。

### 延迟（Lazy）

[`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) 函数接受一个 Lambda 表达式并返回一个 `Lazy<T>` 实例，作为实现了延迟初始化属性的委托：首次调用 `get()` 时会运行传递给 `lazy()` 的 lambda 表达式，并存储结果，之后调用 `get()` 就会直接返回已存储的结果。

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}
```

上面例子的输出为：

```
computed!
Hello
Hello

```

默认情况下，延迟初始化属性的求值是**同步的**（synchronized）：值的计算仅发生在一个线程，其他所有线程都能看到同一个值。如果委托的初始化不需要同步，可以向 `lazy()` 传递 `LazyThreadSafetyMode.PUBLICATION` 参数，这样多个线程可以同时执行。如果你能够确定初始化只会发生在单一线程，可以使用 `LazyThreadSafetyMode.NONE` 模式，这样不保证线程安全，也不会带来线程安全的相关开销。

### 可观察（Observable）

[`Delegates.observable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html) 接受两个参数：初始值和应对修改的处理程序（Handler）。每当对属性进行赋值时，（在赋值完成之后）Handler 都会被调用，Handler 有三个参数，被赋值的属性、属性的旧值和新值：

```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

上面例子的输出为：

```
<no name> -> first
first -> second
```

如果你想要拦截对属性的赋值并“否决（Veto）”它，需要使用 [`vetoable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html) 替代 `observable()`。传递给 `vetoable` 的 Handler 会在赋值发生*之前*被调用。


## 在映射中存储属性

把属性值存储在映射（Map）里也是一种常见用例，比如解析 JSON 或进行其他一些“动态”的操作，此时可以直接使用映射实例本身作为属性的委托。

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

在上面的例子中，构造器接受一个映射：

```kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

委托属性的值取自映射（字符串键对应属性名称）：

```kotlin
println(user.name) // 输出 "John Doe"
println(user.age)  // 输出 25
```

对于 `var` 属性，要使用 `MutableMap` 替代只读的 `Map`：

```kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```


<a name="局部委托属性"></a>
## 局部委托属性（起自 1.1）

局部变量也可以声明为委托属性，举例来说，可以延迟初始化局部变量：

```kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```

`memoizedFoo` 变量的值只会在第一次访问时计算，如果 `someCondition` 失败了，该变量的值就完全不会被计算。


<a name="属性委托的要求"></a>
## 属性委托的要求

这里总结了委托对象的要求。

对于**只读**属性（即 `val`），委托对象必须提供一个名为 `getValue` 的函数，该函数接受如下参数：

- `thisRef`：类型必须与属性拥有者（对于扩展属性，是被扩展的类型）相同，或是其超类，
- `property`：类型必须是 `KProperty<*>` 或其超类。

这个方法必须返回与属性相同的类型（或其子类）。

对于**可变**属性（`var`），委托对象必须*额外*实现一个名为 `setValue` 的函数，该函数接受如下参数：

- `thisRef`：与 `getValue()` 相同，
- `property`：与 `getValue()` 相同。
- 新值：类型必须与属性相同的，或其子类。

`getValue()` 和/或 `setValue()` 可以用委托类的成员函数的形式提供，也可以用扩展函数的形式提供，后者在需要把属性委托给没有实现这些方法的对象时更加方便。这两个函数都必须使用 `operator` 关键字标记。

委托类可以实现标准库中的 `ReadOnlyProperty` 接口或 `ReadWriteProperty` 接口之一，这两个接口包含了委托属性所要求的 `operator` 函数：

```kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```

<a name="转换规则"></a>
### 转换规则

在每一个委托属性背后，Kotlin 编译器都会生成一个隐藏的辅助属性并委托于它。举例来说，对于如下的属性 `prop`，生成的隐藏属性为 `prop$delegate`，`prop` 访问器的代码只是简单地把访问委托给了 `prop$delegate`

```kotlin
class C {
    var prop: Type by MyDelegate()
}

// 下面是编译器生成的代码：
class C {
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

Kotlin 编译器为 `getValue()` 和 `setValue()` 提供了关于 `prop` 的全部必要信息，第一个参数 `this` 指外部类 `C` 的实例，`this::prop` 是 `KProperty` 的反射对象，描述了 `prop` 自身。

注意从 Kotlin 1.1 起才支持在代码中使用如 `this::prop` 的语法指向 [绑定可调用引用](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/reflection.md#绑定函数和属性引用起自-11) 。

### 提供委托（起自 1.1）

通过定义 `provideDelegate` 操作符，可以对属性实现所委托的对象的创建逻辑进行扩展。如果 `by` 右边的对象定义了 `provideDelegate` 作为成员或扩展函数，则在创建属性委托实例的时候，该函数就会被调用。

`provideDelegate` 的一个用例是在属性创建时检查属性的一致性，而不仅仅是在 getter 和 setter 中。

例如，如果你想在绑定委托前检查属性名，可以这样写：

```kotlin
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(
            thisRef: MyUI,
            prop: KProperty<*>
    ): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        // 创建委托
    }

    private fun checkProperty(thisRef: MyUI, name: String) { ... }
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { ... }

class MyUI {
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

`provideDelegate` 的参数和 `getValue` 相同，

- `thisRef`：类型必须与属性拥有者（对于扩展属性的情况，是被扩展的类型）相同，或是其超类，
- `property`：类型必须是 `KProperty<*>` 或其超类。

在创建 `MyUI` 实例时，会为 `MyUI` 的每一个属性调用 `provideDelegate` 方法，并立即进行必要的校验。

如果没有这种对绑定属性和委托进行拦截的机制，要实现类似的功能就必须显式地传递属性名称，并不是很方便：

```kotlin
// 不使用 provideDelegate 功能检查属性名称
class MyUI {
    val image by bindResource(ResourceID.image_id, "image")
    val text by bindResource(ResourceID.text_id, "text")
}

fun <T> MyUI.bindResource(
        id: ResourceID<T>,
        propertyName: String
): ReadOnlyProperty<MyUI, T> {
   checkProperty(this, propertyName)
   // create delegate
}
```

在自动生成的代码中，`provideDelegate` 被用来初始化辅助的 `prop$delegate` 属性，与[前面的代码](#转换规则)相比，对于同样的属性声明 `val prop: Type by MyDelegate()`，这里生成的代码中多出了 `provideDelegate` 方法：

```kotlin
class C {
    var prop: Type by MyDelegate()
}

// 当 provideDelegate 可用时，
// 编译器会生成如下代码
class C {
    // 调用 provideDelegate 以创建额外的 delegate 属性
    private val prop$delegate = MyDelegate().provideDelegate(this, this::prop)
    val prop: Type
        get() = prop$delegate.getValue(this, this::prop)
}
```

注意 `provideDelegate` 仅影响辅助属性的生成，而不影响为 getter 和 setter 生成的代码。
