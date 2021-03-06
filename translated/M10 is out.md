---
title: "[译]M10 is out"
date: 2014-12-17 15:02:00
author: Hadi Hariri
tags:
keywords:
categories: 官方动态
reward: false
reward_title: Have a nice Kotlin!
reward_wechat:
reward_alipay:
source_url: https://blog.jetbrains.com/kotlin/2014/12/m10-is-out/
translator:
translator_url:
---

在庆祝活动开始之前，我们设法发布了 Kotlin 的下一个里程碑，添加了**动态类型**等等。我们来看看 M10 带给我们什么。 <span id =“more-1708”> </span>
## 语言增强

语言的一些改进，特别是：
### 内联函数的 Reified 类型参数

在 M10 之前，我们有时会写这样的代码：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p?.parent
    }
    return p as T
}
```

{% raw %}
<p></p>
{% endraw %}

在这里，我们走一棵树，并使用反射来检查节点是否具有某种类型。这一切都很好，但通话网站看起来有点混乱：

{% raw %}
<p></p>
{% endraw %}

```kotlin
myTree.findParentOfType(javaClass<MyTreeNodeType>())
```

{% raw %}
<p></p>
{% endraw %}

我们实际想要的是简单地将类型*传递给此功能，即调用如下：

{% raw %}
<p></p>
{% endraw %}

```kotlin
myTree.findParentOfType<MyTreeNodeType>()
```

{% raw %}
<p></p>
{% endraw %}

但是，我们需要使用*的泛化泛型*来访问一个函数中的类型，而在 JVM 上，通用的泛型是昂贵的...
幸运的是，Kotlin 有 [内联函数](http://kotlinlang.org/docs/reference/lambdas.html#inline-functions) 现在他们支持**reified**类型参数，所以我们可以这样写：

{% raw %}
<p></p>
{% endraw %}

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p?.parent
    }
    return p as T
}
```

{% raw %}
<p></p>
{% endraw %}

我们使用**reified**修饰符对 type 参数进行限定，现在可以在函数内部访问，几乎就像是普通类一样。由于函数是内联的，因此不需要反射，像**！**这样的正常运算符正在工作。另外，我们可以如上所述调用它：`myTree.findParentOfType＆MyTreeNodeType＆gt;（）`。
虽然在许多情况下可能并不需要反思，但我们仍然可以使用一个重定义的类型参数：`javaClass <t>（）</t>`使我们可以访问它：

{% raw %}
<p></p>
{% endraw %}

```kotlin
inline fun methodsOf<reified T>() = javaClass<T>().getMethods()
 
fun main(s: Array<String>) {
  println(methodsOf<String>().joinToString("\n"))
}
```

{% raw %}
<p></p>
{% endraw %}

正常功能（未标记为`inline`）不能有参数。不具有运行时表示形式的类型（例如，非引用类型参数或类似于`Nothing`的虚构类型）不能用作引用类型参数的参数。
此功能旨在简化传统上依赖于反思的框架中的代码，我们的内部实验表明它的工作正常。
### 检查申报地点差异

Kotlin 有 [申报地点差异](http://kotlinlang.org/docs/reference/generics.html#declaration-site-variance) 从一开始，但编译器长时间缺少通讯员检查。现在他们放在他们的位置：编译器抱怨如果我们在**或**out**中声明一个类型参数为**，但在类体中滥用它：

{% raw %}
<p></p>
{% endraw %}

```kotlin
class C<out T> {
  fun foo(t: T) {} // ERROR
}
```

{% raw %}
<p></p>
{% endraw %}

在这个例子中，由于 T 在 T 中被声明为**out**（即类是*covariant*），我们不允许将它作为`foo 的参数（）`函数，我们只能返回它。
请注意，**私人**声明可以违反方差限制，例如：

{% raw %}
<p></p>
{% endraw %}

```kotlin
class C<out T>(t: T) {
    private var foo: T = t
}
```

{% raw %}
<p></p>
{% endraw %}

虽然`foo`的设置器将 T 作为参数，因此违反了**out**限制，编译器允许这样做，并确保只有同一个实例* `C`可以访问`foo`。这意味着`C`中的以下函数将无法编译：

{% raw %}
<p></p>
{% endraw %}

```kotlin
// inside the class C
private fun copyTo(other: C<T>) {
    other.foo = foo // ERROR: Can not access foo in another instance of C
}
```

{% raw %}
<p></p>
{% endraw %}

这是一个**破坏变化**：以前编译的一些代码可能会中断，但是不能修复它可能会导致运行时异常，所以编译器错误将对你有一些价值<img alt =“:)”class =“wp-smiley”data-recalc-dims =“1”src =“https://i2.wp.com/blog.jetbrains.com/kotlin/wp-includes/images/smilies/ simple-smile.png？w = 640＆amp; ssl = 1“style =”height：1em; max-height：1em“
### 类型推断支持使用方差

类型参数推断已被改进以适应 [使用方差](http://kotlinlang.org/docs/reference/generics.html#type-projections) 更舒适现在您可以调用通用功能，例如一个投影类型的`reverseInPlace（）`，例如`Array＆lt; out Number＆gt;`：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun example(a: Array<out Number>) {
    a.reverseInPlace()
}
```

{% raw %}
<p></p>
{% endraw %}

其中`reverseInPlace`定义如下：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun <T> Array<T>.reverseInPlace() {
    fun (i in 0..size() / 2) {
        val tmp = this[i]
        this[i] = this[size - i - 1]
        this[size - i - 1] = tmp
    }
}
```

{% raw %}
<p></p>
{% endraw %}

最初由罗斯·泰特（Ross Tate）提出的基本机制 [“混合场地差异”](http://www.cs.cornell.edu/~ross/publications/mixedsite/) 。
### Varargs 转换为投影阵列

另一个**突破性变化**的形式是对一个晦涩的修复，但有时候 [相反](https://youtrack.jetbrains.com/issue/KT-5534) [烦人的](https://youtrack.jetbrains.com/issue/KT-2163) 问题：当我们有一个使用`String？`的变量的函数时，我们真的希望能够将`String`的数组传递给它，不是吗？在 M10 之前是不可能的，因为 T 的 vararg 被编译为`Array＆lt; T＆gt;`，现在它们被编译为`Array＆lt; out T＆gt;`，并且以下代码工作：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun takeVararg(vararg strings: String?) { ... }
 
val strs = array("a", "b", "c")
takeVararg(*strs)
```

{% raw %}
<p></p>
{% endraw %}

## JavaScript 改进和更改

JavaScript 在此版本中获得重要更新，支持动态类型。
### 动态支持

有时与动态语言交谈的最佳方式是动态的。这就是为什么我们引入了软键盘*动态*，这样我们可以将类型声明为动态的。目前，这仅在定位 JavaScript 而不是 JVM 时才受支持。
当与 JavaScript 进行互操作时，我们现在可以使用函数作为参数，或返回动态类型

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun interopJS(obj: dynamic): dynamic {
   ...
   return "any type"
}
```

{% raw %}
<p></p>
{% endraw %}

我们将在单独的博客文章中详细介绍动态更多细节以及使用场景和限制。对于技术性见 [规格文件](https://github.com/JetBrains/kotlin/blob/master/spec-docs/dynamic-types.md) 。
### 新注释

我们添加了一系列注释，使 JavaScript 互操作更简单，特别是 nativeInvoke，nativeGetter*和*nativeSetter*。
如果使用`nativeInvoke`注释了一个函数`bar`，它的调用`foo.bar（）`将被转换为`foo（）`在 JavaScript 中。例如：

{% raw %}
<p></p>
{% endraw %}

```kotlin
class Foo {
   nativeInvoke
   fun invoke(a: Int) {}
}
 
val f = Foo()
f(1) // translates to f(1), not f.invoke(1)
f.invoke(1) // also translates to f(1)
```

{% raw %}
<p></p>
{% endraw %}

同样的方式，我们可以使用 NativeGetter*和*nativeSetter*来获取 JavaScript 中的索引访问权限：

{% raw %}
<p></p>
{% endraw %}

```kotlin
native("Array")
class JsArray<T> {
   nativeGetter
   fun get(i: Int): T = noImpl
   nativeSetter
   fun set(i: Int, v: T): Unit = noImpl
}
```

{% raw %}
<p></p>
{% endraw %}

没有`native *`注解，调用`get`和`set`（包括按惯例完成的那些，例如`a [i] = j <代码>与`a.set（i，j）`相同）被翻译成`a.get（...）`和`a.set（.. 。）`，但是注释如上所述，它们在 JavaScript 中被转换为方括号运算符：

{% raw %}
<p></p>
{% endraw %}

```kotlin
a[0] // translates to a[0], not a.get(0)
a.get("first") // translates to a["first"]
a[2] = 3 // translates to a[2] = 3
```

{% raw %}
<p></p>
{% endraw %}

在以下情况下，我们可以使用这些注释：

* 非扩展成员函数的本机声明，
* 顶级扩展功能。

### Kotlin.js 输出突破变化

以前，在创建新项目时，将在名为*scripts*的文件夹中创建 kotlin.js 运行时。从 M10 开始，这个文件是在第一次编译时创建的，并且被放置在输出文件夹（默认为*out*）中。这提供了一个更容易的部署场景，因为库和项目输出现在位于同一根文件夹下。
### kotlin-js 编译器的新 no-stdlib 选项 - 打破变化

我们现在为 kotlin-js 编译器提供一个命令行选项，即*no-stdlib*。没有指定此选项，编译器使用捆绑的标准库。这是 M9 行为的改变。
### js 代码

我们现在可以直接在 Kotlin 代码中输出 JavaScript 代码

{% raw %}
<p></p>
{% endraw %}

```kotlin
js("var value = document.getElementById('item')")
```

{% raw %}
<p></p>
{% endraw %}

它将代码提供为参数，并将其直接注入到编译器生成的 AST JavaScript 代码中。
如您所见，我们在此版本中为 JavaScript 添加了大量新的改进，我们将在单独的文章中更详细地介绍这些改进。
## Java 互操作

### [platformStatic]属性

现在，我们可以将属性标记为`[platformStatic]`，以便它们的访问者从 Java 可视为静态方法。
### 对象中的静态字段

任何对象的属性现在都会产生静态字段，以便它们可以轻松地从 Java 消费，即使不需要使用*platformStatic*注释进行装饰。
### JNI 和[本地]

Kotlin 现在通过`[native]`注释支持 JNI，在`kotlin.jvm`包中定义（参见规范文档 [这里](https://github.com/JetBrains/kotlin/blob/master/spec-docs/jvm-native-flag-support.md) 。）。要声明本机方法，只需将注释放在其上：

{% raw %}
<p></p>
{% endraw %}

```kotlin
import kotlin.jvm.*
import kotlin.platform.*
 
class NativeExample {
    native fun foo() // native method
 
    class object {
        platformStatic native fun bar() // static native method
    }
}
```

{% raw %}
<p></p>
{% endraw %}

这是一个 [例](https://github.com/ligee/kotlin-ndk-samples/blob/master/hello-jni/src/com/example/hellojni/HelloJni.kt) 使用 Android 和 NDK 的原生声明。
## IntelliJ IDEA 改进

IntelliJ IDEA 领域的一些改进包括：
### 混合项目增量汇编

我们已经增强了增量编译，并且使用 M10，它现在支持 Kotlin 和 Java 代码之间的依赖。现在，当您更改 Java 代码时，您的 Kotlin 代码的相关部分将被重新编译。作为提醒，增量编译在 Kotlin 编译器选项下激活。
### HotSwap 在调试器中固定

现在，当我们在调试的时候重新编译 Kotlin 代码，它顺利地被重新加载到了难民程序中。
### 评估表达：完成改进

在调试会话期间，在评估表达式时，根据需要自动添加转换。例如，当从*任何*下载到特定类型时。

{% raw %}
<p><img alt="Completion Casts" class="aligncenter size-full wp-image-1716" data-recalc-dims="1" src="https://i0.wp.com/blog.jetbrains.com/kotlin/files/2014/12/completion.png?resize=564%2C126&amp;ssl=1"/></p>
{% endraw %}

### 复制参考

我们现在可以获得任何 Kotlin 符号的完整参考，就像我们一样 [IntelliJ IDEA 在 Java 代码中](https://www.jetbrains.com/idea/help/cutting-copying-and-pasting.html) {% raw %}
<p><img alt="Copy Reference" class="aligncenter size-full wp-image-1711" data-recalc-dims="1" src="https://i2.wp.com/blog.jetbrains.com/kotlin/files/2014/12/copy-reference-no-retina.png?resize=457%2C269&amp;ssl=1"/></p>
{% endraw %}

### 


从类和包的使用创建

现在，使用创建可用于类和包，这对 TDD 工作流程有重大贡献。即使您不在做 TDD，它也会减少创建新元素的摩擦。

{% raw %}
<p><img alt="Create Class from Usage" class="aligncenter size-full wp-image-1713" data-recalc-dims="1" src="https://i1.wp.com/blog.jetbrains.com/kotlin/files/2014/12/create-class.png?resize=421%2C108&amp;ssl=1"/></p>
{% endraw %}

### 泛型改变签名

变更签名重构现在在基类功能被提升为使用泛型的情况下支持泛型。实质上，在下面的情况

{% raw %}
<p></p>
{% endraw %}

```kotlin
open class Base<T> {
    open fun baseMethod<K>(t: T, k: K) {}
}
 
class Derived<X>: Base<List<X>> {
    override fun baseMethod<Y>(a: List<X>, b: Y) {}
}
```

{% raw %}
<p></p>
{% endraw %}

如果*Base.baseMethod*的签名被更改为*派生的签名，那么将其更改为*baseMethod＆lt; T＆gt;（t：List＆lt; T＆gt;，k：K？）。碱性方法*适当地更改为*＆gt; baseMethod＆lt; Y＆gt;（a：List＆lt; Y＆gt;，b：List＆lt; X＆gt;？
### 完成改进

完成项目订购已得到改进，现在突出显示立即成员。智能完成现在可以找到预期类型的​​继承者。完成表现严重改善。
### 可运行的对象

现在，您可以为 IDE 运行一个声明为`[platformStatic]`主要功能的对象：

{% raw %}
<p></p>
{% endraw %}

```kotlin
import kotlin.platform.*
 
object Hello {
    platformStatic fun main(args: Array<String>) {
        println("Hello")
    }
}
```

{% raw %}
<p></p>
{% endraw %}

只需右键单击该对象，然后选择*运行...*
### 编辑器中的代码覆盖率突出显示

如果您运行 Kotlin 代码与覆盖，编辑器现在标记覆盖和未覆盖的行（仅适用于 IntelliJ IDEA 14）。
### JavaScript 项目配置

IDE 插件现在可以自动配置 Maven 项目以使用 Kotlin / JS。另外，如果您有一个过时的 Kotlin 版本的运行时库，IDE 会要求您进行更新，现在您可以选择使用插件分发中的库，而不是将其复制到项目中。
## 概要

要安装 M10，请更新 IntelliJ IDEA 14（或更早版本）中的插件，并且一如以往，您可以在我们的插件库中找到该插件。您也可以从中下载独立的编译器 [发行页面](https://github.com/JetBrains/kotlin/releases/tag/build-0.10.4) 。
