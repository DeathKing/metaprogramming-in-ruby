# 实用元编程方法

　　本章节将介绍一系列的元编程实用方法，使读者对元编程有一个更为具体的认识。其中一些技术，诸如反射机制，已经有很多文章介绍过了，读者可以根据自身的情况进行选择是否跳过。

## 内省、反射

>　　一说编写元程序的语言称之为元语言。被操纵的程序的语言称之为目标语言。一门编程语言同时也是自身的元语言的能力称之为反射或者自反。
>
> ——摘自维基百科[元编程](http://zh.wikipedia.org/wiki/%E5%85%83%E7%BC%96%E7%A8%8B)条目

　　在Ruby中，你完全有能力在运行时查看类或对象的信息。我们可以使用`class`、 `instance_methods`、  `intance_variables`等方法来达到目的。我们将这种技术称为**内省（Introspection）**或者**反射（Reflection）**。请考虑下面的代码：

```ruby
class Rubyist
  def what_does_he_do
    @person = 'A Rubyist'
    'Ruby programming'
  end
end

an_object = Rubyist.new
puts an_object.class # => Rubyist
puts an_object.class.instance_methods(false) # => what_does_he_do
an_object.what_does_he_do
puts an_object.instance_variables # => @person
```

　　`respond_to?`方法是反射机制中另一个有用的方法。使用`respond_to?`方法，可以提前知道对象是否能够处理你想要交予它执行的信息。所有的对象都有此方法，使用`respond_to?`方法，你可以确定对象是否能使用指定的方法。

```ruby
obj = Object.new
if obj.respond_to?(:program)
  obj.program
else
  puts "Sorry, the object doesn't understand the 'program' message."
end
```

## send

　　`send`是`Object`类的实例方法。`send`方法的第一个参数是你期望对象执行的方法的名称。可以是一个**字符串（String）**或者**符号（Symbol）**，但是我们更喜欢使用符号。剩余的参数就直接传递给所指定的方法。

```ruby
class Rubyist
  def welcome(*args)
    "Welcome " + args.join(' ')
  end
end

obj = Rubyist.new
puts(obj.send(:welcome, "famous", "Rubyists"))   # => Welcome famous Rubyists
```

　　使用`send`方法，你所想要调用的方法就顺理成章的变成了一个普通的参数。你可以在运行时，直至最后一刻自由决定到底调用哪个方法。

```ruby
class Rubyist
end

rubyist = Rubyist.new
if rubyist.respond_to?(:also_railist)
  puts rubyist.send(:also_railist)
else
  puts "No such information available"
end
```
　　上述代码展示了如果`rubyist`对象知道如何处理`also_railist`方法，那么他将会进行处理。

　　你可以通过`send`方法调用任何方法，即使是私有方法。

```ruby
class Rubyist
  private
    def say_hello(name)
      "#{name} rocks!!"
    end
end

obj = Rubyist.new
puts obj.send(:say_hello, 'Matz')
```

## define_method

　　Module#define_method是Module类实例的私有方法。因此define_method方法仅能由类或者模块使用。你可以通过define_method动态的在receiver中定义实例方法。而你仅需要传递需要定义的方法的名字，以及一个代码块（block），就如下面演示的那样：

```ruby
class Rubyist
  define_method :hello do |my_arg|
    my_arg
  end
end

obj = Rubyist.new
puts(obj.hello('Matz')) # => Matz
```

## method_missing

　　当Ruby使用look-up机制找寻方法时，如果方法不存在，那么Ruby将会在原receiver中自行调用一个叫做`method_missing`的方法。`method_missing`方法会以符号的形式传递被调用的那个不存在的方法的名字，以数组的形式传递调用时的参数，以及原调用中传递的块。`method_missing`是由`Kernel`模块提供的方法，因此任意对象都有此方法。`Kernel#method_missing`方法能响应`NoMethodError`错误。重载`method_missing`方法允许你对不存在的方法进行处理。

```ruby
class Rubyist
  def method_missing(m, *args, &block)
    puts "Called #{m} with #{args.inspect} and #{block}"
  end
end

Rubyist.new.anything # => Called anything with [] and
Rubyist.new.anything(3, 4) { something } # => Called anything with [3, 4] and #<Proc:0x02efd664@tmp2.rb:7>
```

　　关于`method_missing`，[ihower](http://ihower.tw/blog/about)给出了一个漂亮的例子：

```ruby
car = Car.new
car.go_to_taipei
# go to taipei
car.go_to_shanghai
# go to shanghai
car.go_to_japan
# go to japan

class Car
  def go(place)
    puts "go to #{place}"
  end

  def method_missing(name, *args)
    if name.to_s =~ /^go_to_(.*)/
      go($1)
    else
      super
    end
  end
end
```

　　这个例子出自于他在Ruby Conf China 2010上的讲义[《如何设计出漂亮的Ruby API》](http://ihower.tw/blog/archives/4797)中，你能够在他的个人博客中看到相关介绍，不过这个例子只出现在[讲义的幻灯片](http://www.slideshare.net/ihower/designing-ruby-apis)中。

> **注意**
>
> `method_missing`方法的效率不甚理想，对效率敏感的项目尽量要避免使用此方法。尽管`method_missing`的确很强力。

## remove_method和undef_method

　　想要移除已存在的方法，你可以在一个打开的类的**作用域（Scope）**内使用`remove_method`方法。即使是父类以及父类的父类等先祖中有同名的方法，那些方法也不会被移除。而相比之下，`undef_method`会阻止任何对指定方法的访问，无论该方法是在对象所属的类中被定义，还是其父类及其先祖类。

```ruby
class Rubyist
  def method_missing(m, *args, &block)
    puts "Method Missing: Called #{m} with #{args.inspect} and #{block}"
  end

  def hello
    puts "Hello from class Rubyist"
  end
end

class IndianRubyist < Rubyist
  def hello
    puts "Hello from class IndianRubyist"
  end
end

obj = IndianRubyist.new
obj.hello # => Hello from class IndianRubyist

class IndianRubyist
  remove_method :hello  # removed from IndianRubyist, but still in Rubyist
end
obj.hello # => Hello from class Rubyist

class IndianRubyist
  undef_method :hello   # prevent any calls to 'hello'
end
obj.hello # => Method Missing: Called hello with [] and
```

## eval

　　`Kernel`模块提供了一个叫做`eval`的方法，该方法用于执行一个用字符串表示的代码。`eval`方法可以计算多行代码，使得将整个程序代码嵌入到字符串中并执行成为了可能。`eval`方法很慢，在执行字符串前最好对其预先求值。不过，糟糕的是，`eval`方法会变得十分危险。如果外部数据通过`eval`传递的话，你就可能会遭遇一个安全漏洞，因为这些数据可能含有任意的代码，并且你的程序将会执行它。现在，`eval`方法是在万般无奈的情况下才被选择的。

```ruby
str = "Hello"
puts eval("str + ' Rubyist'") # => "Hello Rubyist"
```

　　关于`eval`方法，苏小脉给出了下面的建议：

> 　　一般来说，能避免`eval`就尽量避免，因为`eval` 有额外的“分析时”开销（将字符串作为源代码进行词法、文法分析），而这个“剖析时”却又是在程序“运行时”进行的。把不需要惰性求值的表达式预先进行及早求值，能避免一些分析时开销。如果可能的话，用`instance_exec`，或`instance_eval` 带块的形式，也比直接在字符串上求值好。
>
> ——苏小脉在[如果用这种方式来构造一些复杂的对象呢？](http://bbs.66rpg.com/thread-165322-1-1.html)上的发言

　　而关于`eval`方法的安全性漏洞，Dave Thomas在他的著作Programming Ruby（[英文页面](http://www.rubycentral.com/pickaxe/taint.html)）中列举了一个十分有趣的例子：

> 　　Walter Webcoder有一个非常棒的想法：设计一个Web算数页面。该页面是含有一个文本域以及按钮的简单Web表单，并被各种各样的非常酷的数学链接和横幅广告包围，使得看起来丰富多彩。用户输入一个算术表达式到文本域中，并按下按钮，然后结果就会被显示出来。一夜之间，世界上所有计算器都变得无用了；Walter大大获利，然后他退休并把他的余生用于收集车牌号。
>
> Walter认为实现这样一个计算器很容易。他可以用Ruby的`CGI`库访问表单域中的内容，再用`eval`方法把字符串当做表达式来求值。
> ```ruby
> require 'cgi'
>
> cgi = CGI::new("html4")
>
> # Fetch the value of the form field "expression"
> expr = cgi["expression"].to_s
>
> begin
>  result = eval(expr)
> rescue Exception => detail
>  # handle bad expressions
> end
>
> # display result back to user...
> ```
>
> Walter把这个应用程序放到网上才几秒钟，来自Waxahachie的一个12岁小孩在表单中输入了`system("rm")`，随他的计算机上的文件一起，Walter的美梦一下子破灭了。
>
> Walter得到了一个重要的教训：所有的外部数据都是有危险的。不要让它们靠近那些可能改动你的系统的接口。在这个案例中，表单中的内容是外部数据，而对`eval`的调用正是一个安全漏洞。
>
> ——Programming Ruby 中文第二版，Dave Thomas，Chad Fowler，Andy Hunt著


## instance_eval, module_eval, class_eval

　　`instance_eval`，`module_eval`和`class_eval`是`eval`方法的特殊形式。

### instance_eval

　　`Object`类提供了一个名为`instance_eval`的公开方法，该方法可被一个实例调用。他提供了操作对象的实例变量的途径。可以使用字符串向此方法传递参数或者传递一个代码块。

```ruby
class Rubyist
  def initialize
    @geek = "Matz"
  end
end
obj = Rubyist.new

# instance_eval可以操纵obj的私有方法以及实例变量

obj.instance_eval do
  puts self # => #<Rubyist:0x2ef83d0>
  puts @geek # => Matz
end
```
　　通过`instance_eval`传递的代码块使得你可以在对象内部操作。你可以在对象内部肆意操纵，不再会有任何数据是私有的！`instance_eval`亦可用于添加类方法。

```ruby
class Rubyist
end

Rubyist.instance_eval do
  def who
    "Geek"
  end
end

puts Rubyist.who # => Geek
```
　　还记得我们在之前匿名类的讲述（代码清单第四条）么？这个例子在这里被再一次的使用。


### module_eval, class_eval

　　`module_eval`和`class_eval`方法用于模块和类，而不是对象。`class_eval`是`module_eval`的一个别名。`module_eval`和`class_eval`可用于从外部检索类变量。

```ruby
class Rubyist
  @@geek = "Ruby's Matz"
end
puts Rubyist.class_eval("@@geek") # => Ruby's Matz
```

　　`module_eval`和`class_eval`方法亦可用于添加类或模块的实例方法。尽管名字上两个方法时不同的，但他们的功能是相同的，并且都可以在模块或者类上使用。

```ruby
class Rubyist
end
Rubyist.class_eval do
  def who
    "Geek"
  end
end
obj = Rubyist.new
puts obj.who # => Geek
```

> 备注
>
> 当作用于类时，`class_eval`将会定义实例方法，而`instance_eval`定义类方法。


## class_variable_get, class_variable_set

　　添加或查询一个类变量，`class_variable_get`方法和`class_variable_set`方法都可以被使用。`class_variable_get`方法需要一个代表变量名称的符号作为参数，并返回变量的值。`class_variable_set`方法也需要一个代表变量名称的符号作为参数，同时也要求传递一个值，作为欲设定的值。

```ruby
class Rubyist
  @@geek = "Ruby's Matz"
end

Rubyist.class_variable_set(:@@geek, 'Matz rocks!')
puts Rubyist.class_variable_get(:@@geek) # => Matz rocks!
```

## class_variables

　　如果你想知道一个类中有哪些类变量，我们可以使用`class_varibles方法`。他返回一个**数组（Array）**，以**符号（Symbol）**的形式返回类变量的名称。

```ruby
class Rubyist
  @@geek = "Ruby's Matz"
  @@country = "USA"
end

class Child < Rubyist
  @@city = "Nashville"
end
print Rubyist.class_variables # => [:@@geek, :@@country]
puts
p Child.class_variables # => [:@@city]
```

　　你可以从程序的输出中观察到`Child.class_variables`输出的是在`Child`类中定义的类变量（`@@city`）。`Child`类没有从父类中继承类变量（`@@geek`, `@@country`）。


## instance_variable_get, instance_variable_set

　　我们可以使用`instance_variable_get`方法查询实例变量的值。

```ruby
class Rubyist
  def initialize(p1, p2)
    @geek, @country = p1, p2
  end
end
obj = Rubyist.new('Matz', 'USA')
puts obj.instance_variable_get(:@geek) # => Matz
puts obj.instance_variable_get(:@country) # => USA
```

　　类比于`class_variable_set`，你可以使用`instance_variable_set`来设置一个对象的实例变量的值。

```ruby
class Rubyist
  def initialize(p1, p2)
    @geek, @country = p1, p2
  end
end

obj = Rubyist.new('Matz', 'USA')
puts obj.instance_variable_get(:@geek) # => Matz
puts obj.instance_variable_get(:@country) # => USA
obj.instance_variable_set(:@country, 'Japan')
puts obj.inspect # => #<Rubyist:0x2ef8038 @country="Japan", @geek="Matz">
```

　　这样做的好处就是，你不需要使用`attr_accessor`等方法为访问实例变量建立接口。


## const_get, const_set

　　类似的，`const_get`和`const_set`用于操作常量。`const_get`返回指定常量的值：

```
puts Float.const_get(:MIN) # => 2.2250738585072e-308
```

　　`const_set`为指定的常量设置指定的值，并返回该对象。如果常量不存在，那么他会创建该常量，就是下面示范的那样：

```
class Rubyist
end
puts Rubyist.const_set("PI", 22.0/7.0) # => 3.14285714285714
```

　　因为`const_get`返回常量的值，因此，你可以使用此方法获得一个类的名字并为这个类添加一个新的实例化对象的方法。这样使得我们有能力在运行时创建类并实例化其实例。

```ruby
# Let us call our new class 'Rubyist'
# (we could have prompted the user for a class name)
class_name = "rubyist".capitalize
Object.const_set(class_name, Class.new)
# Let us create a method 'who'
# (we could have prompted the user for a method name)
class_name = Object.const_get(class_name)
puts class_name # => Rubyist
class_name.class_eval do
  define_method :who do |my_arg|
    my_arg
  end
end
obj = class_name.new
puts obj.who('Matz') # => Matz
```
