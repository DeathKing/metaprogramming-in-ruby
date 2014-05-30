# 元编程实战

## 问题1

　　此问题改编自Dave Thomas的屏播[Episode 5: Nine Examples of Metaprogramming](http://pragprog.com/screencasts/v-dtrubyom/the-ruby-object-model-and-metaprogramming)。

　　众所周知，RubyLearnin.org的Core Ruby课程已经开办8周了。每周我们都有一个满分10分的测验。8周结束后，学生可以知道他的分数百分比。例如，有一个学生，在过去的8周里，他的得分情况为：5、10、10、10、10、10、10、10。那么，他的得分百分比为93.75%。

　　问题描述：每一批Core Ruby学习班有成百上千的学生。让我们假设我们有一个可以计算这个百分比并返回对应的值的Ruby方法。


### 现存的类和方法

　　让我们先来看看已存在的类和方法，并修改它，来解决上述问题。

```ruby
class Result
  def total(*scores)
    percentage_calculation(*scores)
  end

  private

  def percentage_calculation(*scores)
    puts "Calculation for #{scores.inspect}"
    scores.inject {|sum, n| sum + n } * (100.0/80.0)
  end
end

r = Result.new
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
```

　　上述代码中，我们定义了`Result`类以及一个`total`方法。`total`方法用于列举每个学生的成绩。`score`代表了学生在课程的8次竞赛中获得的成绩。私有方法`percentage_calculation`用于计算等分率。为了测试如此，我们调用`total`方法四次。前两次和后两次调用时分别采用相同的数组，运行程序，我们得到下面的输出：

```ruby
Calculation for [5, 10, 10, 10, 10, 10, 10, 10]
93.75
Calculation for [5, 10, 10, 10, 10, 10, 10, 10]
93.75
Calculation for [10, 10, 10, 10, 10, 10, 10, 10]
100.0
Calculation for [10, 10, 10, 10, 10, 10, 10, 10]
100.0
```

　　观察输出，我们意识到我们调用了`total`方法四次，这也意味着我们调用`percentage_calculation`方法四次。我们将要尝试减少调用`percentage_calculation`方法的次数。

### 常规做法

　　减少对`percentage_calculation`方法的调用的一种途径是用某种方法存放之前计算出的数据。对此，我们需要定义一个叫做`MemoResult`的子类，该子类拥有一个叫做`@mem`的`Hash`类实例变量。代码如下：

```ruby
class Result
  def total(*scores)
    percentage_calculation(*scores)
  end

  private

  def percentage_calculation(*scores)
    puts "Calculation for #{scores.inspect}"
    scores.inject {|sum, n| sum + n } * (100.0/80.0)
  end
end

class MemoResult < Result
  def initialize
    @mem = {}
  end
  def total(*scores)
    if @mem.has_key?(scores)
      @mem[scores]
    else
      @mem[scores] = super
    end
  end
end

r = MemoResult.new
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
```

　　`Hash`类提供了`has_key?`方法用于检查某个散列是否拥有指定的键。在上述程序中，如果`has_key?`返回`true`，那么我们就直接使用`@mem`中存放的值，否则我们将调用`percentage_calculation(*scores)`重新计算改值并存放在`@mem`中。输出如下：

```ruby
Calculation for [5, 10, 10, 10, 10, 10, 10, 10]
93.75
93.75
Calculation for [10, 10, 10, 10, 10, 10, 10, 10]
100.0
100.0
```
　　观察上述输出，我们不难得出，在第二次、第四次调用`r.total`的时候，我们直接使用的存储值。

### 使用Class.new和define_method的做法

　　上面使用的`MemoResult`类与其父类精密相连。为了避免这样，我们使用目前学过的Ruby元编程知识动态创建这个子类。

　　我们需要编写一个接受两个参数的方法`mem_result`：一个参数为父类的名字，另一个参数为方法的名字（而`mem_result`方法会返回定义好的类的名字）。下面是代码：

```ruby
class Result
  def total(*scores)
    percentage_calculation(*scores)
  end

  private

  def percentage_calculation(*scores)
    puts "Calculation for #{scores.inspect}"
    scores.inject {|sum, n| sum + n } * (100.0/80.0)
  end
end

def mem_result(klass, method)
  mem = {}
  Class.new(klass) do
    define_method(method) do |*args|
      if mem.has_key?(args)
        mem[args]
      else
        mem[args] = super
      end
    end
  end
end

r = mem_result(Result, :total).new
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
```
　　输出如下：

```ruby
Calculation for [5, 10, 10, 10, 10, 10, 10, 10]
93.75
93.75
Calculation for [10, 10, 10, 10, 10, 10, 10, 10]
100.0
100.0
```

　　代码`Class.new(klass)`以给定的`klass`为父类，创建了一个匿名类。块被用作类的体，包含了该类中的方法。而`define_method`定义了`method`所代表的方法（也就是`mem_result`的第二个参数）。

> 注意
>
> 我们并没有编写`initialize`方法和使用实例变量`@mem`。相反地，我们使用的是局部变量`mem`，这是因为块是一个闭包，而局部变量`mem`在块内部是有效的。

### 使用匿名类的做法

```ruby
class Result
  def total(*scores)
    percentage_calculation(*scores)
  end

  private

  def percentage_calculation(*scores)
    puts "Calculation for #{scores.inspect}"
    scores.inject {|sum, n| sum + n } * (100.0/80.0)
  end
end

r = Result.new

# Anonymous class on object
def r.total(*scores)
  @mem ||= {}
  if @mem.has_key?(scores)
    @mem[scores]
  else
    @mem[scores] = super
  end
end

puts r.total(5,10,10,10,10,10,10,10)
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
```

### 使用即时创建匿名类的做法

```ruby
class Result
  def total(*scores)
    percentage_calculation(*scores)
  end

  private

  def percentage_calculation(*scores)
    puts "Calculation for #{scores.inspect}"
    scores.inject {|sum, n| sum + n } * (100.0/80.0)
  end
end

def mem_result(obj, method)
  obj.class.class_eval do
    mem ||= {}
    define_method(method) do |*args|
      if mem.has_key?(args)
        mem[args]
      else
        mem[args] = super
      end
    end
  end
end

r = Result.new
mem_result(r, :total)

puts r.total(5,10,10,10,10,10,10,10)
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
```

　　上述的代码中，我们编写了新的`mem_result`方法。该方法的第一个参数`obj`接收一个对象，用于建立此对象的匿名类。该方法的第二个参数`method`接受一个方法，用于指明要在匿名类中创建的方法名字。

　　在此之前，我们已经使用`define_method`即时地创建了一个方法。问题是，`define_method`只能对类或模块起作用，而我们这里处理的是对象。因此，我们使用`obj.class`来获得对象所属的类，然后对类使用`class_eval`和`define_method`方法在该类中添加一个实例方法`method`。现在，我们来运行一下代码并检查输出。

```
result.rb:21:in `total': super: no superclass method `total' (NoMethodError)
  from result.rb:30
```

　　代码没有运行。

　　代码`mem[args] = super`尝试在匿名类中调用`Result`类中的`total`方法。但问题是，我们是在`Result`类中直接定义的`total`方法。我们说过，`obj.class`返回的是`Result`，所以这并不起效。我们需要做得是创建一个匿名类，并将这个`total`方法放到这个匿名类中。同时，我们的匿名类应该是`Result`类的子类。

　　让我们像下面一样创建需要的匿名类。

```ruby
anon = class << obj
  self
end
```

　　上述代码中的`self`返回了我们需要的匿名类，并被变量`anon`所引用。大多数的Ruby程序员会像下面这样把这些代码写作一行，以表示他们正创建鬼魂类。

```ruby
anon = class << obj; self; end
```

　　有了匿名类后，我们应该对其使用`class_eval`方法，就像下面这样：

```
class Result
  def total(*scores)
    percentage_calculation(*scores)
  end

  private

  def percentage_calculation(*scores)
    puts "Calculation for #{scores.inspect}"
    scores.inject {|sum, n| sum + n } * (100.0/80.0)
  end
end

def mem_result(obj, method)
  anon = class << obj; self; end
  anon.class_eval do
    mem ||= {}
    define_method(method) do |*args|
      if mem.has_key?(args)
        mem[args]
      else
        mem[args] = super
      end
    end
  end
end

r = Result.new
mem_result(r, :total)

puts r.total(5,10,10,10,10,10,10,10)
puts r.total(5,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
puts r.total(10,10,10,10,10,10,10,10)
```

　　代码正常运行，并返回希望的结果。

## 问题2

　　本例根据Hal Fulton的文章[An Exercise in Metaprogramming with Ruby(http://an%20exercise%20in%20metaprogramming%20with%20ruby/)改编。

　　假设我们有两个CSV（comma-separated values，数据使用逗号分隔的）文件，其头部有一些描述性的文字，如下所示：

　　文件：location.txt
```
name,country
"Matz", "USA"
"Fabio Akita", "Brazil"
"Peter Cooper", "UK"
```

　　文件：twitter.txt
```
twitterid,url
"AkitaOnRails","http://www.akitaonrails.com/"
"peterc","http://www.petercooper.co.uk/"
```

　　首先，我们建立一个名为datawrapper.rb的文件，并建立一个类。我们会调用`DataWrapper`类并定义一个名为`wrap`的类方法，此方法请求一个用于标识文件名的参数，并据此建立一个类。上面的两个文件的第一行都是由逗号分隔的属性名（attribute names）。因此，我们想要把这些文件当做是数据数组，我们将要读取这些数据，并以数组的形式存放。

```ruby
# file: datawrapper.rb
class DataWrapper
   def self.wrap(file_name)
     data = File.new(file_name)
     header = data.gets.chomp
     data.close
     puts header # => name,country
     # in the end we return the class name
  end
end
```

　　接着，我们编写一个小程序名为testdatawrapper.rb。尝试着读取location.txt文件。

```ruby
#testdatawrapper.rb
require 'datawrapper'
DataWrapper.wrap("location.txt")
```

　　回到datawrapper.rb程序，我们需要新建一个类，并给他适当的名字：

```ruby
# file: datawrapper.rb
class DataWrapper
  def self.wrap(file_name)
    data = File.new(file_name)
    header = data.gets.chomp
    data.close
    class_name = File.basename(file_name, ".txt").capitalize
    klass = Object.const_set(class_name, Class.new)
    klass # we return the class name
  end
end
```

　　变量`klass`指代的是我们新建的类。如果`file_name`参数所指向的文件为“location.txt”，那么新建立得类名会被命名为`Location`。

　　再次运行修改后的程序。

```ruby
#testdatawrapper.rb
require 'datawrapper'
data = DataWrapper.wrap("location.txt") # Capture return value and
puts data # => Location
```

　　然后就是大展身手的时候了。文件第一行读入的是表名（List Name）。我们只需要使用`split`方法即可快速读入。 修改后的datawrapper.rb如下：

```ruby
# file: datawrapper.rb
class DataWrapper
  def self.wrap(file_name)
    data = File.new(file_name)
    header = data.gets.chomp
    data.close
    class_name = File.basename(file_name, ".txt").capitalize
    klass = Object.const_set(class_name, Class.new)
    # get attribute names
    names = header.split(",")
    p names # => ["name", "country"]
    klass # we return the class name
  end
end
```

　　现在，在我们新建立的类的上下文中使用`class_eval`。此时，我们会定义一个`initialize`方法。并且，我们应该建立一个`to_s`方法，使得我们能够将其输出，记得使用`alias`关键字将`to_s`方法作为`inspect`方法的同义词。修改后的datawrapper.rb程序如下：

```ruby
# file: datawrapper.rb
class DataWrapper
  def self.wrap(file_name)
    data = File.new(file_name)
    header = data.gets.chomp
    data.close
    class_name = File.basename(file_name, ".txt").capitalize
    klass = Object.const_set(class_name, Class.new)
    # get attribute names
    names = header.split(",")
    klass.class_eval do
      attr_accessor *names
      define_method(:initialize) do |*values|
        names.each_with_index do |name, i|
          instance_variable_set("@"+name, values[i])
        end
      end
      define_method(:to_s) do
        str = "<#{self.class}:"
        names.each {|name| str << " #{name}=#{self.send(name)}" }
        str + ">"
      end
      alias_method :inspect, :to_s
    end
    klass # we return the class name
  end
end
```

　　下一步，建立一个类方法，用于读取整个文本，并返回一个代表文本中内容的素组对象。类方法的建立涉及到了单体类等概念，但此处不需要仔细考察。修改后的代码如下：

```ruby
# file: datawrapper.rb
class DataWrapper
  def self.wrap(file_name)
    data = File.new(file_name)
    header = data.gets.chomp
    data.close
    class_name = File.basename(file_name, ".txt").capitalize
    klass = Object.const_set(class_name, Class.new)
    # get attribute names
    names = header.split(",")
    klass.class_eval do
      attr_accessor *names
      define_method(:initialize) do |*values|
        names.each_with_index do |name, i|
          instance_variable_set("@"+name, values[i])
        end
      end
      define_method(:to_s) do
        str = "<#{self.class}:"
        names.each {|name| str << " #{name}=#{self.send(name)}" }
        str + ">"
      end
      alias_method :inspect, :to_s
    end
    def klass.read
      array = []
      data = File.new(self.to_s.downcase+".txt")
      data.gets  # throw away header
      data.each do |line|
        line.chomp!
        values = eval("[#{line}]")
        array << self.new(*values)
      end
      data.close
      array
    end
    klass # we return the class name
  end
end
```

　　修改testdatawrapper.rb并测试。

```ruby
#testdatawrapper.rb
require 'datawrapper'
klass = DataWrapper.wrap("location.txt")
list = klass.read
list.each do |location|
  puts("#{location.name} is from the #{location.country}")
end
```

　　让我们来看看和location.txt文件完全不同的twitter.txt文件。针对于twitter.txt的测试文件如下：

```ruby
#testdatawrapper.rb
require 'datawrapper'
klass = DataWrapper.wrap("twitter.txt")
list = klass.read
list.each do |twitter|
  puts("#{twitter.twitterid}'s site is #{twitter.url}")
end
```

　　即使我们使用了不同的数据，datawrapper.rb的代码也无需改变！这便是一个Ruby之中的元编程的例子与实践。
