# 前言

## 元编程概览

　　元编程的定义看似是明确的，但却又模棱两可。维基百科上对元编程的定义如下：

> 　　元编程是指某类计算机程序的编写，这类计算机程序编写或者操纵其它程序（或者自身）作为它们的数据，或者在运行时完成部分本应在编译时完成的工作。多数情况下，与手工编写全部代码相比，程序员可以获得更高的工作效率, 或者给与程序更大的灵活度去处理新的情形而无需重新编译。

　　而我也在网上找到了[Free Mind](http://lifegoo.pluskid.org/?p=46)对元编程的简介：

> 　　回到元编程，程序处理程序可以分为“处理其他程序”和“处理自己”，对于前者，有我们熟悉的lex和yacc作为例子。而对于后者，如果再细分，可以分为“宏扩展”、“源代码生成”以及“运行时动态修改”等几种。
>
> 　　宏扩展从最简单的C语言的宏到复杂的Lisp的宏系统，甚至C++的“模板元编程”也可以包含在这一类里面，我在这里对它们进行了一些介绍。
>
> 　　源代码生成则主要是利用编程语言的eval功能，对生成出来的源代码（除了在Lisp这样的语言里面以外，通常是以字符串的方式）进行求值。有一类有趣的程序quine，它们运行的结果就是把自己的源代码原封不动地打印出来，通常要证明你精通某一门语言，为它写一个quine是绝佳的选择了，这里有我搜集的一些相关资料。
>
> 　　最后是运行时修改，像Lisp、Python以及Ruby之类的语言允许在运行时直接修改程序自身的结构，而不用再经过“生成源代码”这一层。当然对源代码进行eval的方法也许是最灵活的，但是它的效率却不怎么样，所以相来说并不是特别流行。这里主要介绍的是这种方式的元编程在Ruby里面的应用，如果对元编程的其他方面感兴趣，前面的几个链接应该都是很好的去处。

　　而元编程技术，最早来自于Lisp。John M. Vlissides在[Pattern Languages of Program Design](http://books.google.com.hk/books?id=QhMocNso2XoC&pg=PA214&lpg=PA214&dq=lisp+%E5%85%83%E7%BC%96%E7%A8%%20%208B&source=bl&ots=T92M3sWWcE&sig=zs9oTXIzkDB-_Y3-rqo1Z4ZgM0c&hl=zh-%20%20CN&ei=fHZQTrvlNsjMrQfZmPSsAg&sa=X&oi=book_result&ct=result&resnum=9&ved=0CFcQ6AEwCA#v=onepage&q=lisp%20%E5%85%83%  E7%BC%96%E7%A8%8B&f=false)一书中写到：

> 　　Lisp社团位于现在称之为“反射编程”（reflective programming）或“元编程”（metaprogramming）——对可编程语言编程的前沿。Smalltalk事实上从20世纪70年代后期就开始支持元类。但它是Lisp专用语言，从Comman和ObjVlisp开始，推动元编程成为主流[Bobrow+86，Cointe87]。早期工作为主对象协议（Metaobject Protocal）[Kiczales+91]打下了基础。主对象协议主要用来推广元编程。商业系统也开始使用元编程，特别是在IBM SOM平台上。

　　而你或许不知道[Meta（源于希腊语）](http://en.wikipedia.org/wiki/Meta)这个前缀在这里的意思是抽象。

## Ruby中的元编程

　　Ruby中的元编程，是可以在运行时动态地操作语言结构（如类、模块、实例变量等）的技术。你甚至于可以在不用重启的情况下，在运行时直接键入一段新的Ruby代码，并执行它。

　　Ruby的元编程，也具有“利用代码来编写代码”的作用。例如，常见的`attr_accessor`等方法就是如此。

## 一点技术上的细节

>　　Lisp通过既可以用于代码也可以用于数据的S表达式（本质上是程序抽象语法树的直接翻译）支持语法层的元编程。Lisp的元编程大量的使用了宏，宏的本质是代码模板。Lisp的这种方式带来的好处是可以在单一的层次上进行编程，代码及数据都以相同的方式表现，唯一的区别在于是否会被估值（evaluate）。然而语法层的元编程模式也有其弊端，用在同一命名空间下运行和估值的代码对两个抽象层次进行操作，会直接导致变量捕获（Variable Capture）和不经意的多次估值这类问题的出现。纵使有标准的Lisp惯用法可以处理这些问题，Lisp程序员仍然是需要学习和考虑更多的东西。
>
>　　Ruby可以通过ParseTree库来完成语法层的自省，ParseTree可以将Ruby源代码翻译成S表达式。使用此库来编写的一个有趣的应用叫做Heckle，它是一个“测试的测试”框架，能够对Ruby代码解析及改变，例如改变字符串或者将'true'和'false'进行来回的调换。其想法是如果测试覆盖率很好，那么对代码任何部分的任何变更都应该导致单元测试的失败。
>
>　　与语法自省相对应的一种更高层次的自省叫做语义自省，既通过语言更高层次的数据结构对程序进行探查。在不同的编程语言中语义自省的方式十分不同，在Ruby语言中一般来说都是作用于类及方法层上：创建方法，重写方法，给方法赋予别名（alias）；截取方法调用；操纵继承链。这些技术和语法层自省相比与已有的代码更为正交（相关度更小），因为它们倾向于将已存在的方法视为黑盒而不是修改其内部实现。
>
> ——摘自[里克的自习室](http://railser.cn/blog/what-is-metaprogramming/)


## 更多的参考资料

1. Ruby's Metaprogramming toolbox:http://weare.buildingsky.net/2009/08/25/rubys-metaprogramming-toolbox
2. Metaprogramming in Ruby:http://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/
3. Practical Metaprogramming with Ruby: Storing Preferences:http://www.kalzumeus.com/2009/11/17/practical-metaprogramming-with-ruby-storing-preferences/
4. The Ruby Object Model and Metaprogramming:http://pragprog.com/screencasts/v-dtrubyom/the-ruby-object-model-and-metaprogramming
5. Metaprogramming Ruby: class_eval and instance_eval:http://jimmycuadra.com/posts/metaprogramming-ruby-class-eval-and-instance-eval
6. Metaprogramming in Ruby:http://rubyrogues.com/metaprogramming-in-ruby/
