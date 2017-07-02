# 接口

Kotlin 中的接口和 Java 8 非常相似，可以拥有抽象方法和方法实现。接口与抽象类的区别在于接口不能储存状态。接口可以具有属性，但属性需要是抽象的，或者仅提供访问器的实现。

使用 `interface` 关键字声明接口：

```kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```


## 实现接口

类或 `object` 可以实现一个或多个接口：

```kotlin
class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```


## 接口中的属性

在接口中可以声明属性，但接口中的属性必须是抽象的，或者提供访问器的实现。接口中声明的属性不具有支持字段（Backing Field），所以接口中声明的访问器不能引用属性自身。

```kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```


## 解决覆盖冲突

如果在超类列表中声明了多个类型，有时会出现继承到同一个方法的多个实现的情况，例如：

```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```

接口 `A` 和 `B` 都具有方法 `foo()` 和 `bar()`，且都实现了 `foo()`，但只有 `B` 实现了 `bar()`（`A` 中的 `bar()` 没有标记为抽象，因为如果接口中的函数没有函数体，则该函数默认是抽象的）。现在，如果我们由 `A` 派生一个具体类 `C`，我们显然需要覆盖 `bar()` 并提供实现。

<a name="注1返回"></a>
但是，如果我们从 `A` 和 `B` 派生出 `D`，我们就要实现从多个接口中继承的全部的方法，以明确 `D` 到底要如何实现他们。这一规则同时适用于继承来的只有单个实现方法（`bar()`），以及继承来的具有多个实现的方法（`foo()`）。[【注 1】](#注1)


---

<a name="注1"></a>【注 1】尽管只有 `B` 给出了 `bar()` 的默认实现，`D` 还是需要实现 `bar()`。如果只有 `B` 又定义了一个方法 `foobar()` 并给出了默认实现，那么 `D` 就不需要实现 `foobar()`，因为 `D` 只能从 `B` 继承 `foobar()` 的实现，不存在歧义。
