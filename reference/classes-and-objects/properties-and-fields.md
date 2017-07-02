# 属性和字段

## 声明属性

Kotlin 中的类可以具有属性。可以使用 `var` 关键字声明可变（Mutable）属性，或者使用 `val` 关键字声明只读（Read-Only）属性。

```kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String? = ...
    var zip: String = ...
}
```

直接通过属性名就可以引用属性，就像使用 Java 的字段一样：

```kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // Kotlin 中没有 new 关键字
    result.name = address.name // 调用了访问器
    result.street = address.street
    // ...
    return result
}
```

## Getter 和 Setter

声明属性的完整语法为：

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

其中的初始化器（`property_initializer`）、getter 和 setter 都是可选的。如果属性类型能够通过初始化（或者 getter 的返回值类型，如下所示）推断出来，则属性类型也可以省略。

例如：

```kotlin
var allByDefault: Int? // 错误: 需要显式初始化，使用默认 getter 和 setter
var initialized = 1 // 类型为 Int, 使用默认 getter 和 setter
```

声明只读属性的完整语法与可变属性有两点区别：只读属性使用 `val` 关键字而不是 `var`，且不允许有 setter：

```kotlin
val simple: Int? // 类型为 Int, 默认 getter, 必须在构造器中初始化
val inferredType = 1 // 类型为 Int, 默认 getter
```

我们可以在属性声明中编写自定义的访问器，就像常规的方法一样，如自定义 getter:

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

也可以自定义 setter:

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 解析字符串并对其他属性赋值
    }
```

按照惯例，setter 的参数名为 `value`，但你也可以选用任意的名称。

从 Kotlin 1.1 开始，如果属性的类型可以从 getter 推断出来，就可以在声明属性时省略类型：

```kotlin
val isEmpty get() = this.size == 0  // 类型为 Boolean
```

如果想要在不修改默认实现的情况下，修改访问器的可见性或者添加注解，可以只定义访问器的名称而不给出实现：

```kotlin
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

### 支持字段

Kotlin 的类没有字段，但有时在使用自定义访问器时，需要有一个支持字段（Backing Field）。为此，Kotlin 提供了自动创建的支持字段，可以使用 `field` 标识符来访问：

```kotlin
var counter = 0 // the initializer value is written directly to the backing field
    set(value) {
        if (value >= 0) field = value
    }
```

`field` 标识符只能在属性的访问器中使用。

只有当属性使用了一个以上默认实现的访问器，或者自定义访问器使用 `field` 标识符引用了支持字段时，才会为该属性创建支持字段。

例如，下面的情况就不会创建支持字段：

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

### 支持属性

如果“隐式支持字段”的方案不能满足需要，那么还可以使用支持属性。

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // 可以推断出类型参数
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

从各方面来看，上面代码就像 Java（声明私有字段并提供 getter）。使用私有属性和默认的 getter、setter 是经过优化的，不会引入函数调用的开销。


## 编译时常量

如果属性的值在编译期间就是已知的，可以使用 `const` 关键字把它标记为编译时常量（Compile Time Constant），此类属性需要满足如下要求：

- 为顶层属性，或者是 `object` 的成员
- 初始化为 `String` 或基本类型
- 没有自定义 getter

编译时常量可以在注解中使用：

```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```


## 延迟初始化属性

通常来说，具有非空（Non-Null）类型的属性必须在构造器中初始化，但这通常不是很方便，比如属性可以通过依赖注入或单元测试的配置方法完成初始化。在这些情况下，并不能在构造器中提供非空初始化，但又希望在类中引用属性时避免空引用检查。

为了处理这种情况，可以使用 `lateinit` 标识符来标记属性：

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // 直接解引用
    }
}
```

`lateinit` 只能用于在类中（而不是主构造器中）定义的 `var` 属性，且该属性不能具有自定义的 getter 或 setter。该属性的类型必须为非空，且不能是基本类型（Primitive Type）。

在 `lateinit` 属性初始化前访问该属性时，会抛出一个特殊的异常，指明当前访问的属性尚未初始化。


## 覆盖属性

见[覆盖属性](https://blog.nex3z.com/2017/06/04/kotlin-reference-classes-inheritance/#Overriding_Properties)。


## 委托属性

最常见的一类属性仅从支持字段中进行读取（或写入），而自定义 getter 和 setter 可以实现属性的任意行为。在这两种场景之间，还有其他常见的特定模式，比如延迟初始化值、根据 Key 读取 Map，读取数据库、在被访问时通知监听者等。

这些行为可以通过[委托属性（Delegated Property）](https://blog.nex3z.com/2017/06/10/kotlin-reference-delegated-properties/)以库的形式实现。
