# 泛型

Kotlin 的类可以具有类型参数，就像 Java 一样：

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

一般来说，要创建这种类的实例，需要提供类型参数：

```kotlin
val box: Box<Int> = Box<Int>(1)
```

但如果类型参数可以被推断出来，例如通过构造器参数或其他途径，则可以省略类型参数：

```kotlin
val box = Box(1) // 1 的类型是 Int, 所以编译器可以知道我们指的是
```


## 变型

通配符是 Java 类型系统中最复杂部分之一（见 [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)），Kotlin 中没有通配符，取而代之的是声明处变型（Declaration-Site Variance）和类型投射（Type Projection）。

首先，让我们来思考为什么 Java 需要这些神秘的通配符。[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 在 “第28条：利用有限制通配符来提升 API 的灵活性（Item 28: Use bounded wildcards to increase API flexibility）”中解释了需要通配符的原因。首先，Java 中的泛型类型是**不可变的（Invariant）**，这意味着 `List<String>` **不是** `List<Object>` 的子类。如果 `List` 是可变的，那它就和 Java 的数组有一样的问题，如下面的代码可以通过编译，但在运行时抛出异常：

```java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! 下面的问题源自这里，Java 禁止此种用法！
objs.add(1); // 这里我们向 String 列表中放入了一个 Integer
String s = strs.get(0); // !!! ClassCastException: Cannot cast Integer to String
```

所以 Java 禁止这种的用法，以保证运行时的安全。但这种限制也带来一些额外的影响，比如对于 `Collection` 接口的 `addAll()` 方法，直觉上我们会认为该方法的签名如下：

```java
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

<a name="注1返回"></a>
然而使用上面的声明，会导致无法进行如下（完全安全的）简单操作：

```java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from); // !!! 对于上面声明的 addAll 将无法编译：
                   //     Collection<String> 不是 Collection<Object> 的子类
}
```
[【注 1】](#注1)

（我们在 Java 中吸取了惨痛了教训，见 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 第25条：列表优先于数组 Item 25: Prefer lists to arrays）

所以 `addAll()` 实际使用的签名如下：

```java
// Java
interface Collection<E> …… {
  void addAll(Collection<? extends E> items);
}
```

这里的**通配符类型参数** `? extends E` 表示该方法接受 `E` 的子类的集合作为参数，而不仅限于 `E` 本身的集合。这意味着我们可以安全地从集合中**读取**类型 `E` 的元素（集合中的所有元素都是 `E` 的子类的实例），但不能对该集合进行**写入**，因为不知道哪些对象符合“`E` 的未知子类”的要求。以不能写入为代价，我们得到了希望的效果： `Collection<String>` *表现为了* `Collection<? extends Object>` 的子类。用专业术语来讲，使用 `extends` 限定（上边界）的通配符使得类型是协变的（Covariant）。

<a name="注2返回"></a>
理解上面工作原理的关键其实很简单，如果只能从集合中**取出**元素，把 `String` 的集合中的元素以 `Object` 的类型读取出来是没有问题的。反过来说，如果只能往集合中**放入**元素，把 `String` 的实例放进 `Object` 的集合也是可以的。在 Java 中，`List<? super String>` 可以作为 `List<Object>` 的超类[【注 2】](#注2)。

上面的后一种情况称为**逆变性（Contravariance）**。对于 `List<? super String>`，只可以调用它以 `String` 作为参数的方法（例如可以调用 `add(String)`、`set(int, String)`），但如果调用它返回 `List<T>` 中的 `T` 的方法，得到的会是 `Obejct` 而不是 `String`。

Joshua Bloch 把仅能从中**读取**的对象称为**生产者（Producer）**，仅能向其中**写入**的对象称为**消费者（Consumer）**，他建议，“为了保证最大的灵活性，在生产者或消费者的输入参数上使用泛型”，并提出了如下的助记方式：

**PECS 表示生产者 Extend，消费者 Super。（PECS stands for Producer-Extends, Consumer-Super.）**

注意：如果使用了生产者对象，如 `List<? extends Foo>`，就不能调用能修改该对象的 `add()` 或 `set()`，但这并不意味着该对象是**不可变的（Immutable）**，例如依然可以调用 `clear()` 来清空整个列表，因为 `clear()` 没有参数。通配符（或者其他形式的变型）只能确保**类型安全（Type Safety）**，不可变性（Immutability）则完全是另一码事。

### 声明处变型

假设我们有一个泛型接口 `Source<T>`，它没有任何以 `T` 作为参数的方法，只有一个返回 `T` 的方法：

```java
// Java
interface Source<T> {
  T nextT();
}
```

这样，把 `Source<String>` 的实例的引用保存到 `Source<Object>` 类型的变量是完全安全的，因为 `Source<T>` 不具有消费者方法。但 Java 并不知道这一点，所以会禁止如下的用法：

```java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! Java 中不允许这种写法
  // ...
}
```

为了解决这一问题，我们必须把 `objects` 的类型声明为 `Source<? extends Object>`，但这并没有什么实际意义，因为我们依然可以像之前一样调用 `objects` 上的所有方法。在这种场景下，像 `Source<? extends Object>` 这种更复杂的声明并没有带来什么实际价值，但编译器并不知道这一点。

在 Kotlin 中，可以通过**声明处变型（Declaration-Site Variance）** 向编译器说明上面提到的情况：我们可以对**类型参数** `T` 进行标注，确保它仅能用于 `Source<T>` 成员的返回（生产）类型，而从不被消费。为了达到这样的效果，我们只需使用 `out` 修饰符：

```kotlin
abstract class Source<out T> {
    abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // 这是是可以的, 因为 T 是一个 out 参数
    // ...
}
```

一般的规则是：如果类 `C` 的类型参数 `T` 被声明为 `out`，则 `T` 仅能出现在 `C` 的成员的**输出（Out）**位置，由此 `C<Base>` 可以安全地作为 `C<Derived>` 的超类。

使用专业术语来说，类 `C` 对于参数 `T` 是**协变的（Covariant）**，或者说 `T` 是一个**协变的**类型参数。可以把 `C` 想象为 `T` 的一个**生产者**，而不是 `T` 的**消费者**。

`out` 修饰符称为变型注解（Variance Annotation），又由于它用在类型参数的声明处，故称之为**声明处变型（Declaration-Site Variance）**。这与 Java 使用通配符来实现类型协变的**使用处变型（Use-Site Variance）**相反。

除了 `out`，Kotlin 还提供了与之互补的标注 `in`，`in` 使得类型参数成为**逆变的（Contravariant）**：仅能被消费，不能被生产。`Comparable` 是一个典型的逆变类：

```kotlin
abstract class Comparable<in T> {
    abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 的类型是 Double，它是 Number 的子类
    // 因此，我们可以把 x 赋给 Comparable<Double> 型的变量
    val y: Comparable<Double> = x // OK！
}
```

我们相信 `in` 和 `out` 非常直白（它们已经成功地在 C# 中使用了很久），并不需要特别记忆如上面 PECS 的口诀，甚至可以换个更好的表述：

**存在性转换：消费者 `in`，生产者 `out`。（[The Existential](https://en.wikipedia.org/wiki/Existentialism) Transformation: Consumer in, Producer out! :-)）**


## 类型投射

### 使用处变型：类型投射

把类型参数 `T` 声明名为 `out` 可以很轻松地避免使用处的子类化问题，但有的类**并不能**仅限于返回 `T`，`Array` 就是一个很好的例子：

```kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { /* ... */ }
    fun set(index: Int, value: T) { /* ... */ }
}
```

`T` 既不能是协变的，也不能是逆变的。这就带来了不灵活的地方，如对于下面的函数：

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```

这个函数用于把一个数组的元素复制到另个数组，在实际使用的时候：

```kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 
copy(ints, any) // 错误: 期望 (Array<Any>, Array<Any>)
```

这里就遇到了和开头类似的问题，`Array<T>` 对于 `T` 是**不可变的**，因此 `Array<Int>` 不是 `Array<Any>` 的子类。原因依旧在于 `copy()` 可能干坏事，比如说，它可能会尝试往 `from` 里写入一个 `String`，而如果我们传入的 `from` 是 `Int` 型的数组，则会在运行时抛出 `ClassCastException`。

我们需要确保 `copy()` 不会干坏事，即禁止在 `copy()` 中对 `from` 进行写入，可以使用如下的方式：

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
 // ...
}
```

这里出现的就是**类型投射（Type Projection）**，我们表明 `from` 不仅是一个数组，而且是一个受限（**投射的**）的数组：我们只能调用其返回类型参数 `T` 的方法，这里我们只能调用 `get()`。这是 Kotlin 中的**使用处变型（Use-Site Variance）**，对应 Java 的 `Array<? extends Object>`，但稍微简单一些。

也可以使用 `in` 来投射类型：

```kotlin
fun fill(dest: Array<in String>, value: String) {
    // ...
}
```

`Array<in String>` 对应 Java 的 `Array<? super String>`，如可以向 `fill()` 传递 `CharSequence` 数组或 `Object` 数组。

### 星号投射

有时候并不知道类型参数的信息，但依旧想要确保类型安全。安全地方法是定义一个泛型类型的投射，使得该泛型的任意具体实例都是该投射的子类。

为此，Kotlin 提供了**星号投射（Star-Projection ）**的语法：

- 对于 `Foo<out T>`，其中 `T` 是一个具有上边界 `TUpper` 的协变类型参数，`Foo<*>` 相当于 `Foo<out TUpper>`。这意味着当 `T` 未知时，可以安全地从 `Foo<*>` 中读取 `TUpper` 类型的值。
- 对于 `Foo<in T>`，其中 `T` 是一个逆变类型参数，`Foo<*>` 相当于 `Foo<in Nothing>`。这意味着如果 `T` 是未知的，就不能向 `Foo<*>` 安全地写入任何值。
- 对于 `Foo<T>`，其中 `T` 是一个具有上边界 `TUpper` 的不变类型参数，`Foo<*>` 在读取时相当于 `Foo<out TUpper>`，在写入时相当于 `Foo<in Nothing>`。

如果泛型有多个类型参数，则各个类型参数可以分别投射，比如对于 `interface Function<in T, out U>`，可以有如下的星号投射：

- `Function<*, String>` 表示 `Function<in Nothing, String>`;
- `Function<Int, *>` 表示 `Function<Int, out Any?>`;
- `Function<*, *>` 表示 `Function<in Nothing, out Any?>`.

星号投射类似于 Java 的原始类型（Raw Type），但它是安全的。


## 泛型函数

函数也可以像类一样具有类型参数，类型参数放在函数名前：

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString() : String {  // extension function
    // ...
}
```

调用泛型函数时，要在调用处的函数名**之后**指定类型参数：

```kotlin
val l = singletonList<Int>(1)
```


## 泛型约束

对于一个给定类型参数，可以使用泛型约束来对可以替代它的类型的集合进行限制。

### 上边界

最常用的约束是指定**上边界（Upper Bound）**，对应 Java 的 `extends`：

```kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
    // ...
}
```

冒号后面的类型是**上边界**，表示只有 `Comparable<T>` 的子类可以替代 `T`，如：

```kotlin
sort(listOf(1, 2, 3)) // OK。Int 是 Comparable<Int> 的子类
sort(listOf(HashMap<Int, String>())) // 错误: HashMap<Int, String> 不是 Comparable<HashMap<Int, String>> 的子类
```

未指定上边界时，默认的上边界是 `Any?`，尖括号内只能声明一个上边界，如果同一个类型需要多个上边界，需要使用单独的 `where` 子句：

```kotlin
fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T>
    where T : Comparable,
          T : Cloneable {
  return list.filter { it > threshold }.map { it.clone() }
}
```


---
<a name="注1"></a>【注 1】`String` 是 `Object` 的子类，把 `String` 的对象添加到 `Object` 集合是安全的，但由于泛型是不可变的，`Collection<String>` 不是 `Collection<Object>` 的子类，故上面的代码无法编译。[【返回】](#注1返回)

<a name="注2"></a>【注 2】`? super String` 表示 `String` 的任意超类， `Object` 只是其中一种。[【返回】](#注2返回)
