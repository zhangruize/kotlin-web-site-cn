---
type: doc
layout: reference
category: "Syntax"
title: "函数"
---

# 函数

## 函数声明

Kotlin 中的函数使用 *fun*{: .keyword } 关键字声明

``` kotlin
fun double(x: Int): Int {
    return 2*x
}
```

## 函数用法

调用函数使用传统的方法

``` kotlin
val result = double(2)
```


调用成员函数使用点表示法

``` kotlin
Sample().foo() // 创建类 Sample 实例并调用 foo
```

### 中缀表示法

函数还可以用中缀表示法调用，当

* 他们是成员函数或[扩展函数](extensions.html)
* 他们只有一个参数
* 他们用 `infix` 关键字标注

``` kotlin
// 给 Int 定义扩展
infix fun Int.shl(x: Int): Int {
……
}

// 用中缀表示法调用扩展函数

1 shl 2

// 等同于这样

1.shl(2)
```

### 参数

函数参数使用 Pascal 表示法定义，即 *name*: *type*。参数用逗号隔开。每个参数必须有显式类型。

``` kotlin
fun powerOf(number: Int, exponent: Int) {
……
}
```

### 默认参数

函数参数可以有默认值，当省略相应的参数时使用默认值。与其他语言相比，这可以减少<!--
-->重载数量。

``` kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) {
……
}
```

默认值通过类型后面的 **=** 及给出的值来定义。

覆盖方法总是使用与基类型方法相同的默认参数值。
当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值：

``` kotlin
open class A {
    open fun foo(i: Int = 10) { …… }
}

class B : A() {
    override fun foo(i: Int) { …… }  // 不能有默认值
}
```

### 命名参数

可以在调用函数时使用命名的函数参数。当一个函数有大量的参数或默认参数时这会非常方便。

给定以下函数

``` kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
……
}
```

我们可以使用默认参数来调用它

``` kotlin
reformat(str)
```

然而，当使用非默认参数调用它时，该调用看起来就像

``` kotlin
reformat(str, true, true, false, '_')
```

使用命名参数我们可以使代码更具有可读性

``` kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

并且如果我们不需要所有的参数

``` kotlin
reformat(str, wordSeparator = '_')
```

请注意，在调用 Java 函数时不能使用命名参数语法，因为 Java 字节码并不<!--
-->总是保留函数参数的名称。


### 返回 Unit 的函数

如果一个函数不返回任何有用的值，它的返回类型是 `Unit`。`Unit` 是一种只有一个值——`Unit` 的类型。这个<!--
-->值不需要显式返回

``` kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` 或者 `return` 是可选的
}
```

`Unit` 返回类型声明也是可选的。上面的代码等同于

``` kotlin
fun printHello(name: String?) {
    ……
}
```

### 单表达式函数

当函数返回单个表达式时，可以省略花括号并且在 **=** 符号之后指定代码体即可

``` kotlin
fun double(x: Int): Int = x * 2
```

当返回值类型可由编译器推断时，显式声明返回类型是[可选](#显式返回类型)的

``` kotlin
fun double(x: Int) = x * 2
```

### 显式返回类型

具有块代码体的函数必须始终显式指定返回类型，除非他们旨在返回 `Unit`，[在这种情况下它是可选的](#返回-unit-的函数)。
Kotlin 不推断具有块代码体的函数的返回类型，因为这样的函数在代码体中可能有复杂的控制流，并且返回<!--
-->类型对于读者（有时甚至对于编译器）是不明显的。


### 可变数量的参数（Varargs）

函数的参数（通常是最后一个）可以用 `vararg` 修饰符标记：

``` kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

允许将可变数量的参数传递给函数：

``` kotlin
val list = asList(1, 2, 3)
```

在函数内部，类型 `T` 的 `vararg` 参数的可见方式是作为 `T` 数组，即上例中的 `ts` 变量具有类型 `Array <out T>`。

只有一个参数可以标注为 `vararg`。如果 `vararg` 参数不是列表中的最后一个参数， 可以使用<!--
-->命名参数语法传递其后的参数的值，或者，如果参数具有函数类型，则通过在括号外部<!--
-->传一个 lambda。

当我们调用 `vararg`-函数时，我们可以一个接一个地传参，例如 `asList(1, 2, 3)`，或者，如果我们已经有一个数组<!--
-->并希望将其内容传给该函数，我们使用**伸展（spread）**操作符（在数组前面加 `*`）：

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

## 函数作用域

在 Kotlin 中函数可以在文件顶层声明，这意味着你不需要像一些语言如 Java、C# 或 Scala 那样创建一个类来保存一个函数。此外<!--
-->除了顶层函数，Kotlin 中函数也可以声明在局部作用域、作为成员函数以及扩展函数。

### 局部函数

Kotlin 支持局部函数，即一个函数在另一个函数内部

``` kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

局部函数可以访问外部函数（即闭包）的局部变量，所以在上例中，*visited* 可以是局部变量。

``` kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

### 成员函数

成员函数是在类或对象内部定义的函数

``` kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

成员函数以点表示法调用

``` kotlin
Sample().foo() // 创建类 Sample 实例并调用 foo
```

关于类和覆盖成员的更多信息参见[类](classes.html)和[继承](classes.html#继承)

## 泛型函数

函数可以有泛型参数，通过在函数名前使用尖括号指定。

``` kotlin
fun <T> singletonList(item: T): List<T> {
    // ……
}
```

关于泛型函数的更多信息参见[泛型](generics.html)

## 内联函数

内联函数在[这里](inline-functions.html)讲述

## 扩展函数

扩展函数在[其自有章节](extensions.html)讲述

## 高阶函数和 Lambda 表达式

高阶函数和 Lambda 表达式在[其自有章节](lambdas.html)讲述

## 尾递归函数

Kotlin 支持一种称为[尾递归](https://zh.wikipedia.org/wiki/%E5%B0%BE%E8%B0%83%E7%94%A8)的函数式编程风格。
这允许一些通常用循环写的算法改用递归函数来写，而无堆栈溢出的风险。
当一个函数用 `tailrec` 修饰符标记并满足所需的形式时，编译器会优化该递归，留下一个快速而高效的基于循环的版本。

``` kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

这段代码计算余弦的不动点（fixpoint of cosine），这是一个数学常数。 它只是重复地从 1.0 开始调用 Math.cos，直到结果不再改变，产生0.7390851332151607的结果。最终代码相当于这种更传统风格的代码：

``` kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

要符合 `tailrec` 修饰符的条件的话，函数必须将其自身调用作为它执行的最后一个操作。在递归调用后有更多代码时，不能使用尾递归，并且不能用在 try/catch/finally 块中。目前尾部递归只在 JVM 后端中支持。

