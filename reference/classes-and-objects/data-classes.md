# 数据类

我们经常会创建一些仅用于存储数据的类，在这些类中，通常会根据数据派生出一系列标准的功能。Kotlin 称之为数据类（Data Class），使用 `data` 标记：

```kotlin
data class User(val name: String, val age: Int)
```

编译器会自动根据主构造器中声明的所有属性，为数据类生成以下成员：

- `equals()` / `hashCode()` 对，
- `toString()`，形式为 `"User(name=John, age=42)"`，
- 按顺序对应每一个属性的 [`componentN()` 函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/destructuring-declarations.md)，
- `copy()` 函数（见下）。

如果以上函数已经在类中显式地声明，或者已从基类中继承得到，则不会自动生成该函数。

为了确保生成代码的一致性，并具有有意义的行为，数据类必须满足一下要求：

- 主构造器必须至少有一个参数；
- 主构造器的所有参数必须标记为 `val` 或 `var`；
- 数据类不能为 `abstract`、`open`、`sealed` 或 `inner`；
- （在 1.1 版本之前）数据类只能实现接口。

从 1.1 版本起，数据类可以继承其他类（例子见[密封类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/sealed-classes.md)）。

在 JVM 平台，如果希望生成的类具有一个无参构造器，则主构造器的所有参数必须都有默认值，详见[构造器](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/classes-and-inheritance.md#构造器)。

```kotlin
data class User(val name: String = "", val age: Int = 0)
```


## 复制

我们经常需要把对象复制一份，修改其中的部分属性，而不改变其他属性，`copy()` 函数就是用于这种场景。对于上面的 `User` 类，它的 `copy()` 函数实现如下：

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)    
```

于是我们可以这样写：

```kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```


## 数据类和解构声明

数据类也会生成一系列成分函数（Component Function），以用于[解构声明（Destructuring Declaration）](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/other/destructuring-declarations.md):

```kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // 输出 "Jane, 35 years of age"
```


## 标准数据类

　　标准库提供了 `Pair` 和 `Triple`。但在大多数情况下，最好使用命名数据类，因为有意义的名字和属性可以提供更好的可读性。

