---
title: [译]Kotlin M5 is Out!
date: 2013-02-04 14:30:00
author: Andrey Breslav
tags:
keywords:
categories: 官方动态
reward: false
reward_title: Have a nice Kotlin!
reward_wechat:
reward_alipay:
source_url: https://blog.jetbrains.com/kotlin/2013/02/kotlin-m5-is-out/
---

从现在开始的两个星期内，Kotlin将是一个开源项目的一年。在这个时候，这是一个很大的努力，在社区的巨大帮助下：我们收到了164个拉动请求，这意味着每隔一天的贡献。今天我们又一步迈出了Kotlin M5。本博客文章涵盖了此版本中引入的更改。
# 概述

M5是一个短暂的里程碑（你应该从这个术语中减去新年的休息时间），但是我们在追踪器中摆脱了144个问题。
许多IDE子系统得到了改进，包括JUnit转轮，从Java搜索Kotlin类，更好地对无效外部注释进行诊断，新图标和对Darcula配色方案的支持：

{% raw %}
<p><a href="https://i2.wp.com/blog.jetbrains.com/kotlin/files/2013/02/Darcula.png"><img alt="" class="aligncenter size-medium wp-image-836" data-recalc-dims="1" sizes="(max-width: 300px) 100vw, 300px" src="https://i2.wp.com/blog.jetbrains.com/kotlin/files/2013/02/Darcula.png?resize=300%2C224&amp;ssl=1" srcset="https://i2.wp.com/blog.jetbrains.com/kotlin/files/2013/02/Darcula.png?resize=300%2C224&amp;ssl=1 300w, https://i2.wp.com/blog.jetbrains.com/kotlin/files/2013/02/Darcula.png?w=965&amp;ssl=1 965w"/></a></p>
{% endraw %}

语言的微小变化包括更好地支持Float文字（现在您可以简单地说1.0，其中需要Float），以及将定位和命名参数混合到函数调用中的能力。
一些变化不是那么谦虚，可能需要你修复现有的代码...
# 软件包类

在旧版本的Kotlin中，每个包含顶级函数或声明的属性的包都被编译为一个名为“namespace”的类，其中顶级声明由静态方法表示。当您在Java中使用了多个这些“命名空间”类时，您遇到了名称冲突：您不能将两个具有相同名称的类导入同一个编译单元。与Kotlin M5包类别命名为相应的包，这给了他们不同的名称，并解决了这个问题。
命名约定如下：package“org.example”获取一个类“org.example.ExamplePackage”。即，我们采用简单的包名称，将其大小写，附加“Package”，并将该类放入包中。到目前为止，它的效果很好。
注意：由于此更改，您的旧版本的kotlin-runtime.jar将无法再工作。编译器将抱怨“不兼容的ABI版本”，IDE将提出用新的运行时jar替换旧的运行时jar。
# 内部类

内部类是一个非静态嵌套类，即它保存对其外部实例的引用。在Java中，嵌套类默认是内部的，如果不希望引用外部类，那么你的类就是静态的。有时它会导致内存泄漏，当有人持有对内部类的实例的引用，而不知道它也包含一个外部实例。
由于M5，Kotlin希望你明确标记内部类，默认情况下嵌套类是“static”。这可能会破坏您现有的代码，在IDE中有一个方便的快速解决方案（只需按下Alt + Enter即可）。

{% raw %}
<p><img alt="" class="aligncenter" data-recalc-dims="1" src="https://i0.wp.com/www.evernote.com/shard/s171/sh/b06bbb46-0577-47f3-a715-f3473e1b4f16/e8cb41d5ccdd6ff192c7647619bf47d5/res/df4fb94b-51ea-4923-8538-ea590dbb5467/Add_inner_modifier-20130204-135715.png.jpg?w=640&amp;ssl=1"/></p>
{% endraw %}

# Java泛型和可空性

泛型是棘手的，它们与可空类型的组合更复杂。在Java中，一切都是可以空的，例如，考虑一个Java方法foo（ArrayList <String>），Kotlin（在M5之前）用于将它看作是ArrayList <String？>，即集合可以是null，其元素可能是也是。这是我们可以做的最安全的事情，但是它被证明是非常不方便的：如果你在Kotlin中有一个ArrayList <String>，你不能传递给foo（）：ArrayList在它的通用参数中是不变的，因此ArrayList < String>不是ArrayList <String？>的子类型。即使使用KAnnotator，也会造成很大的痛苦。
所以我们决定改变一般参数类型的默认策略，并加载ArrayList <String>？在上述情况下。
此更改可能会破坏现有的一些代码。大部分可以通过删除不需要的问号直接固定。如果你想要旧的类型，你可以添加一个外部注释到你的Java定义。
但安全呢？现在Java代码可能会给你一个空值的集合而不是字符串，而你的Kotlin代码将会失败。这可能会发生，但是我们使它失败：Kotlin检查从Java接收到的数据，并提前失败，并出现如下详细的错误消息：

{% raw %}
<p></p>
{% endraw %}

```kotlin
Exception in thread "main" java.lang.IllegalStateException:
    Method specified as non-null returned null: JavaClass.foo
    at _DefaultPackage.main(hello.kt:4)
```

{% raw %}
<p></p>
{% endraw %}

这可能比NPE好多了，也许。对功能参数进行同样的检查：如果有人将非法通过空值的Kotlin功能称为非法，那么它会提早爆发，尽可能地责怪有罪。
# Varargs和函数文字

Kotlin的类型安全的构建器是非常棒的，特别是如果你注意到它们不是一个内置的机制，而只是一个很好的语言功能（主要是扩展功能和高阶函数）的组合。有一件事是在旧版本的Kotlin中打扰构建器的作者：你不能定义一个可以将函数文字作为括号括起来的参数的vararg函数。现在你可以做到：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun css(vararg selectors: Selector, body: Element.() -> Unit) {...}
```

{% raw %}
<p></p>
{% endraw %}

可以叫做

{% raw %}
<p></p>
{% endraw %}

```kotlin
css(TD, _class("data")) {
    background_color = RED
}
```

{% raw %}
<p></p>
{% endraw %}

您也可以使用命名和定位的参数（包括varargs）：

{% raw %}
<p></p>
{% endraw %}

```kotlin
css(TD, body = foo)
```

{% raw %}
<p></p>
{% endraw %}

# 范围

科特林的标准图书馆也在发展，这次我们修改了范围。要提醒您，范围在循环和条件中使用很多：

{% raw %}
<p></p>
{% endraw %}

```kotlin
for (i in 1..10) { /* 1, 2, 3, ..., 10 */ } 
 
if (x in low..high) { /* low <= x <= high */ }
```

{% raw %}
<p></p>
{% endraw %}

新范围在内部更一致，适用于下降迭代，平常增量等情况。我们将在本周的另一篇博客文章中提供更多的细节。
# 默认构造函数

Kotlin每个类只允许一个构造函数。在建模数据时，我们经常为构造函数参数使用默认值（毕竟，这只能使一个构造函数实用）：

{% raw %}
<p></p>
{% endraw %}

```kotlin
class Bean(val data: Integer = 0)
```

{% raw %}
<p></p>
{% endraw %}

现在，构造函数更加方便：在生成的字节代码中，此类将获得默认构造函数，即不使用参数的构造函数（使用它们的默认值）。这种情况在使用Java框架（如JAXB）时出现了很多，所以现在Kotlin更加Java友好。
# 结论

您可以从插件库中下载Kotlin M5。它需要IntelliJ IDEA 12（推荐使用最近发布的12.0.3）。
有一个很好的Kolin！