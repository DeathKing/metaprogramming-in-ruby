# 实例变量、方法、类

## 对象的实例变量及方法

　　**实例变量（Instance Variables）**是当你使用它们时，才会被建立的对象。因此，即使是同一个类的实例，也可以有不同的实例变量。

　　从技术层面上来看，一个对象（实例）只是存储了它的实例变量和其所属类的引用。因此，一个对象的实例变量仅存在于对象中，方法（我们称之为**实例方法（Instance Methods）**）则存在于对象所属的类中。这也就是为什么同一个类的实例都共享类中的方法，却不能共享实例变量的原因了。

## 类

+ 类也是对象。
+ 因为类也是一个对象，能应用于对象的皆可运用于类。类和任何对象一样，有它们自己的类，`Class`类即是`Class`类的实例。
+ 与其它的对象一样，类也有方法。对象的方法即是其所属类的实例方法。亦即，任何一个类的方法就是`Class`类的实例方法。
+ 所有的类有共同的祖先`Object`类（都是从`Object`类直接或间接继承而来），而`Object`类又继承自`BasicObject`类，Ruby类的根本。
+ 类名是**常量（Constant）**。

　　下面的代码有助于你理解这些信息：

```ruby
# 对象的方法即为其所属类的实例方法
1.methods == 1.class.instance_methods
#=> true

# 类的“溯源”
N = Class.new
N.ancestors
#=> [N, Object, Kernel, BasicObject]
N.class
#=> Class
N.superclass
#=> Object
N.superclass.superclass
#=> BasicObject
N.superclass.superclass.superclass
#=> nil
```

## 类是开放的

　　在Ruby中，类始终都是开放的。你可以重定义一个类，甚至于像`String`或`Array`这样的标准库中的类。（译注：Ruby 2.0引入了`refine`来限定这种打开类的作用域。）

```ruby
class String
  def writesize
    self.size
  end
end

puts "Tell me my size!".writesize
```
> 注意!
>
> 　　当你打开一个类的时候，一定要万分小心！如果你随意向类中添加方法或数据，你可能会收到许多的BUG，比如，你定义了自己的`captalize()`方法并且漫不经心的覆盖了原`String`类中的`captalize()`方法，你很可能会收到风险。

## 多重initialize方法

　　下面将示范一个类的**重载（Overloading）**。我们编写了一个`Rectangule`类，该类用于将一个矩形呈现在网格上。当你在实例化一个`Rectangule`对象时，你可以使用两种方法：传递矩形的左上、右下的坐标，或者左上点坐标及矩形的宽度、高度。虽然Ruby中每个类只有一个`initialize`方法，但这种做法允许你的一个`initialize`就像两个不同的`initialize`一样。

```ruby
# The Rectangle constructor accepts arguments in either
# of the following forms:
#   Rectangle.new([x_top, y_left], length, width)
#   Rectangle.new([x_top, y_left], [x_bottom, y_right])
class Rectangle
  def initialize(*args)
    if args.size < 2  || args.size > 3
      puts 'Sorry. This method takes either 2 or 3 arguments.'
    else
      puts 'Correct number of arguments.'
    end
  end
end
Rectangle.new([10, 23], 4, 10)
Rectangle.new([10, 23], [14, 13])
```

　　上述代码还不足以编写一个完整的`Rectangule`程序，但却足以演示方法重载是如何实现的。对`initialize`方法的重载可使得其具有处理可变参数的能力。这种技巧不但适用于`initialize`方法，也适用于其他方法！

## 匿名类

　　一个**匿名类（Anonymous Class）**也被称作**单例类（Singleton Class）**，**特征类（Eigenclass）**，**鬼魂类（Ghost Class）**，**元类（Metaclass）**或者**uniclass**。

> 　　关于eigenclass这个名字的由来：大多数人叫它singlton classes，另一部分人叫它metaclasses，意思是the class of a class。但是这些名字都不足以描述它。Ruby之父Matz至今还没有给出一个官方的名字，但是他似乎喜欢叫它eigenclass，eigen这个词来自于德语，意思是one’s own。所以，eigenclass就被翻译为“an object’s own class”。而eigenclass的方法名字，则取了一个比较科学严肃的名字叫Singlton Methods。
> ——参考自[blackanger对Metaprogramming Ruby的笔记(5)]([blackanger对《Metaprogramming Ruby》的笔记(5)](http://book.douban.com/people/blackanger/annotation/4086938/))

> 　　“特征类”是一个很好的命名。这个“特征”和线性代数中“特征值”、“特征向量”的“特征”是一个意思。“特征值”、“特征向量”的名称是由德国数学家 David Hilbert （大卫·希尔伯特）在 1904 年使用并得以流传的，德语单词“Eigen”本意为“自己的”。Eigenclass，也就意味着“自己的类”，用于 Ruby 的“单例类”概念之上十分贴切，因为“单例类”实际上就是一个对象独有的类。
> ——参考自紫苏的[特征类](http://szsu.wordpress.com/2011/01/17/%E7%89%B9%E5%BE%81%E7%B1%BB/)一文

　　Ruby中每个对象都有其自己的匿名类，一个类能拥有方法，但是只能对该对象本身其作用：当我们对一个具体的对象添加方法时，Ruby会插入一个新的匿名类于父类之间，来容纳这个新建立的方法。值得注意的是，匿名类通常是不可见（Hidden）的。它没有名字因此不能像其他类一样，通过一个常量来访问。你不能为这个匿名类实例化一个新的对象。

　　下面展示了建立匿名类的一些方法：

```ruby
# 1
class Rubyist
  def self.who
    "Geek"
  end
end

# 2
class Rubyist
  class << self
    def who
      "Geek"
    end
  end
end

# 3
class Rubyist
end
def Rubyist.who
  "Geek"
end

#4
class Rubyist
end
Rubyist.instance_eval do
  def who
    "Geek"
  end
end
puts Rubyist.who # => Geek

# 5
class << Rubyist
  def who
    "Geek"
  end
end
```
　　上述5段代码，分别定义了`Rubylist.who`方法，该方法返回`"Geek"`。

　　任何时候，一旦你看到如上述标号为#5的代码，`class`关键字后面紧接着`<<`，你就应该确信这里为`<<`右边的对象打开了一个匿名类。

　　你可以参看[Complete Ruby Class Diagram](http://banisterfiend.wordpress.com/2008/11/25/a-complete-ruby-class-diagram/) 一文，文中展示了Ruby 1.8.6中的用户定义类及他们的超类的关系。而下面这幅由Artem S贡献的Ruby层次关系（Ruby 2.0.0）也非常有用：

![Screen](http://rubylearning.com/images/Ruby_2.0.0_Hierarchy.png)
