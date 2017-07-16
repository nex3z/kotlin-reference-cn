# 类型安全建造者

[建造者（Builder）](http://www.groovy-lang.org/dsls.html#_nodebuilder) 的概念在 *Groovy* 社区中颇为流行。建造者允许以半声明式的形式定义数据，常用于[生成 XML](http://www.groovy-lang.org/processing-xml.html#_creating_xml)、[UI 元素布局](http://www.groovy-lang.org/swing.html)、[描述 3D 场景](http://www.artima.com/weblogs/viewpost.jsp?thread=296081)等。

Kotlin 提供的类型检查建造者适用于大多数用例，比 Groovy 中的动态类型的实现更加诱人。

Kotlin 也支持动态类型建造者，以满足其他用例的需要。

## 一个类型安全建造者的例子

考虑下面的代码：

```kotlin
import com.example.html.* // 见下面的声明

fun result(args: Array<String>) =
    html {
        head {
            title {+"XML encoding with Kotlin"}
        }
        body {
            h1 {+"XML encoding with Kotlin"}
            p  {+"this format can be used as an alternative markup to XML"}

            // 带有文字内容属性的元素
            a(href = "http://kotlinlang.org") {+"Kotlin"}

            // 混合内容
            p {
                +"This is some"
                b {+"mixed"}
                +"text. For more see the"
                a(href = "http://kotlinlang.org") {+"Kotlin"}
                +"project"
            }
            p {+"some text"}

            // 由下面的代码生成内容
            p {
                for (arg in args)
                    +arg
            }
        }
    }
```

这是完全合法的 Kotlin 代码，你可以在<a href="http://try.kotlinlang.org/#/Examples/Longer examples/HTML Builder/HTML Builder.kt">这里</a>在线修改并在浏览器中运行这段代码。


## 工作原理

下面会逐步介绍在 Kolint 中实现类型安全的建造者的方法。首先，我们需要定义想要建造的模型，这里我们对 HTML 标签进行建模，只需简单地定义一些列类即可。如 `HTML` 是一个用于描述 `<html>` 标签的类，它定义了如 `<head>` 和 `<body>` 的子元素。（见[下面](#comexamplehtml包的完整定义)的声明）

现在再回过头看前面的代码：

```kotlin
html {
 // ...
}
```

`html` 其实是一个函数，接受一个 [Lambda 表达式](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md)作为其参数，该函数的定义如下：

```kotlin
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```

这个函数有一个名为 `init` 的参数，该参数本身也是一个函数，该函数的类型是 `HTML.() -> Unit`，是一个*带接收者的函数类型*（Function Type with Receiver）。这意味着我们需要向该函数传递一个 `HTML` 类型（*接收者*）的实例，然后在函数内就可以调用该实例的成员。可以使用 `this` 关键字访问接收者：

```kotlin
html {
    this.head { /* ... */ }
    this.body { /* ... */ }
}
```

（`head` 和 `body` 是 `HTML` 的成员函数）

现在，省略掉 `this`，得到的结果已经非常接近建造者了：

```kotlin
html {
    head { /* ... */ }
    body { /* ... */ }
}
```

那么，这个调用到底做了什么呢？来看上面定义的 `html` 函数的内部，它创建了一个新的 `HTML` 实例，然后调用传递给它的函数来进行初始化（在我们的例子中，归结为在 `HTML` 实例上调用 `head` 和 `body`），然后它返回了这个实例。这正是建造者的工作。

`HTML` 类中 `head` 和 `body` 函数的定义和 `html` 类似，唯一的区别在于，它们把所建造的实例添加到外围 `HTML` 实例的 `children` 集合中：

```kotlin
fun head(init: Head.() -> Unit) : Head {
    val head = Head()
    head.init()
    children.add(head)
    return head
}

fun body(init: Body.() -> Unit) : Body {
    val body = Body()
    body.init()
    children.add(body)
    return body
}
```

实际上，这两个函数做的事情相同，我们可以使用泛型的版本 `initTag`：

```kotlin
protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
    tag.init()
    children.add(tag)
    return tag
}
```

于是函数就变得非常简洁：

```kotlin
fun head(init: Head.() -> Unit) = initTag(Head(), init)

fun body(init: Body.() -> Unit) = initTag(Body(), init)
```

之后就可以使用它们来建造 `<head>` 和 `<body>` 标签了。

这里要讨论的另一个问题是如何为标签体添加文字。在开头的例子中，使用了这样的方法：

```kotlin
html {
    head {
        title {+"XML encoding with Kotlin"}
    }
    // ...
}
```

简单来说，我们只是把字符串放到了标签体内，然后在开头加了一个 `+`，形成了一个调用 `unaryPlus()` 前缀操作的函数调用。该操作是由 `TagWithText` 抽象类（`Title` 的父类）的扩展函数 `unaryPlus()` 定义的：

```kotlin
fun String.unaryPlus() {
    children.add(TextElement(this))
}
```

所以，这里前缀 `+` 的作用是把字符串包装进一个 `TextElement` 实例，然后把它添加到 `children` 集合，成为了标签树的一部分。

上面的内容都定义在开头例子的顶部导入的 `com.example.html` 包中，在最后可以看到该包的完整定义。


## 作用域控制：@DslMarker（起自 1.1）

<a name="注1返回"></a>
使用 DSL 时，有时会遇到在上下文中存在过多可被调用的函数的情况。由于我们可以在 Lambda 中调用所有隐式接收者的方法，因此可能会导致不一致的结果，就像在 `head` 中又调用了 `head`[【注 1】](#注1)：

```kotlin
html {
    head {
        head {} //应当被禁止
    }
    // ...
}
```

在这个例子中，第 3 行处我们只希望能使用最近的隐式接收者 `this@head` （第 2 行的 `head`）的成员，而第 3 行的 `head` 作为更外层的接收者 `this@html` （第 1 行的 `html`）的成员，应当被禁止调用。

Kotlin 1.1 引入了一个特殊的机制来控制接收者的作用域，以解决这个问题。

为了让编译器对作用域进行控制，必须为在 DSL 中使用的所有接收者的类型添加同样的标记注解。举例来说，我们为 HTML 建造者声明了注解 `@HTMLTagMarker`：

```kotlin
@DslMarker
annotation class HtmlTagMarker
```

使用 `@DslMarker` 注解的注解类称为 DSL 标记（DSL Marker）。

在我们的 DSL 中，所有的标签类都继承自同一个超类 `Tag`，只要使用 `@HtmlTagMarker` 注解超类，之后 Kotlin 编译器就会认为所有的子类都有该注解：

```kotlin
@HtmlTagMarker
abstract class Tag(val name: String) { ... }
```

我们不需要使用 `@HtmlTagMarker` 注解 `HTML` 或 `Head` 类，因为它们的超类已经被注解了：

```kotlin
class HTML() : Tag("html") { ... }
class Head() : Tag("head") { ... }
```

在添加了这个注解之后，Kotlin 编译器就知道哪些隐式的接收者是同一个 DSL 的一部分，从而仅允许调用最近接收者的成员：

```kotlin
html {
    head {
        head { } // 错误: 外部接收者的成员
    }
    // ...
}
```

需要注意的是，调用更外层的接收者仍是可行的，但必须显式地指定接收者：

```kotlin
html {
    head {
        this@html.head { } // 允许
    }
    // ...
}
```


## com.example.html 包的完整定义

下面是 `com.example.html` 包的具体定义（仅包含上面用到的元素），它建造了一个 HTML 树，重度使用了[扩展函数](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/classes-and-objects/extensions.md) 和 [带接收者的 Lambda](https://github.com/nex3z/kotlin-reference-cn/blob/master/reference/functions-and-lambdas/lambdas.md#带接收者的函数字面值)。

注意 `@DslMarker` 注解仅从 Kotlin 1.1 起可用。

```kotlin
package com.example.html

interface Element {
    fun render(builder: StringBuilder, indent: String)
}

class TextElement(val text: String) : Element {
    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent$text\n")
    }
}

@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
abstract class Tag(val name: String) : Element {
    val children = arrayListOf<Element>()
    val attributes = hashMapOf<String, String>()

    protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }

    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent<$name${renderAttributes()}>\n")
        for (c in children) {
            c.render(builder, indent + "  ")
        }
        builder.append("$indent</$name>\n")
    }

    private fun renderAttributes(): String {
        val builder = StringBuilder()
        for ((attr, value) in attributes) {
            builder.append(" $attr=\"$value\"")
        }
        return builder.toString()
    }

    override fun toString(): String {
        val builder = StringBuilder()
        render(builder, "")
        return builder.toString()
    }
}

abstract class TagWithText(name: String) : Tag(name) {
    operator fun String.unaryPlus() {
        children.add(TextElement(this))
    }
}

class HTML : TagWithText("html") {
    fun head(init: Head.() -> Unit) = initTag(Head(), init)

    fun body(init: Body.() -> Unit) = initTag(Body(), init)
}

class Head : TagWithText("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title : TagWithText("title")

abstract class BodyTag(name: String) : TagWithText(name) {
    fun b(init: B.() -> Unit) = initTag(B(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun a(href: String, init: A.() -> Unit) {
        val a = initTag(A(), init)
        a.href = href
    }
}

class Body : BodyTag("body")
class B : BodyTag("b")
class P : BodyTag("p")
class H1 : BodyTag("h1")

class A : BodyTag("a") {
    var href: String
        get() = attributes["href"]!!
        set(value) {
            attributes["href"] = value
        }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```


---
<a name="注1"></a>【注 1】`head` 是 `html` 的方法，但在作为 `head` 参数的 Lambda 中，仍可以访问外层 `html` 的 `head` 方法。[【返回】](#注1返回)
