# 嵌套类

<a name="注1返回"></a>
类可以嵌套进其他的类中：

```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

[【注 1】](#注1)


## 内部类

可以把类标记为 `inner`，这样该类就可以方位外部类的成员。内部类会携带外部类对象的引用：

```kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

关于在内部类中消除 `this` 歧义的方法，请参考[限定的 `this` 表达式](https://blog.nex3z.com/2017/06/19/kotlin-reference-expression/) 。


## 匿名内部类

使用[对象表达式（Object Expression）](https://kotlinlang.org/docs/reference/object-declarations.html#object-expressions)可以创建匿名内部类的实例：

```kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
                                                                                                            
    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

　　如果对象是一个函数式 Java 接口（只有一个抽象方法的 Java 接口）的实例，可以通过以接口类型作为前缀的 Lambda 表达式来创建：

```kotlin
val listener = ActionListener { println("clicked") }
```


---
<a name="注1"></a>【注 1】Kotlin 的嵌套类默认不携带外部类的引用，相当于 Java 的静态嵌套类。[【返回】](#注1返回)