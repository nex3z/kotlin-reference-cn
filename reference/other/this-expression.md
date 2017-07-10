# This 表达式

使用 `this` 表达式表示当前的*接收者*（Receiver）：

- 在[类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/classes-and-inheritance.md)的成员中，`this` 指的是该类的当前的对象。
- 在[扩展函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md) 和 [带接收者的函数字面值](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md#带接收者的函数字面值) 中，`this` 指的是在点（`.`）左边传递的*接受者*参数。

如果 `this` 不具有限定词，则它指的是*自身所在的最内层的作用域*。如果要引用外层作用域，则需要使用*标签限定符*（Label Qualifier）：

## 限定的 this

如果想要访问外层作用域（如[类](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/classes-and-inheritance.md)、[扩展函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md)或有标签的[带接收者的函数字面值](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md#带接收者的函数字面值)）的 `this`，需要使用 `this@label`，其中 `@label` 是 `this` 来源作用域的[标签](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/basics/returns-and-jumps.md)：

```kotlin
class A { // 隐式标签 @A
    inner class B { // 隐式标签 @B
        fun Int.foo() { // 隐式标签 @foo
            val a = this@A // A 的 this
            val b = this@B // B 的 this

            val c = this // foo() 的接收者, 一个 Int
            val c1 = this@foo // foo() 的接收者, 一个 Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit 的接收者
            }


            val funLit2 = { s: String ->
                // foo() 的接收者，因为包围的 Lambda 表达式没有接收者
                val d1 = this
            }
        }
    }
}
```
