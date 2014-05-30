# 方法的调用

　　当你在调用某一个方法的时候，Ruby会完成下面的步骤：

+ 找到这个方法，我们把这个过程称作**方法查找（method lookup）**；
+ 执行这个方法，为了执行这个方法，Ruby需要一个叫做`self`的伪变量；

## 方法的查找

　　要理解Ruby的方法查找，你需要了解下面两个概念：**接受者（receiver）**和**祖先链（ancestors chain）**。接受者就是方法的调用者。例如，对调用`an_object.display()`来说，`an_object`就是接受者。而关于祖先链，请考察任何一个Ruby类的内部。想象一下从一个类移动到其父类，然后再移动到父类的父类，直到到达`Object`类（默认的父类），然后继续移动，最终到达`BasicObject`类（Ruby层级模型中的根）。你遍历这些类所走过的路径就是“祖先链”（祖先链中也包括模块）。

　　因此，为了查找方法，Ruby首先进入receiver所属的类，并以此为起始，沿着祖先链不断前进，直到找到目标方法。这种行为也被称作**“向右一步，再向上（one step to the right, then up）”**规则：向右一步，进入reciver所属的类，然后再向上查找祖先链，直到找到目标方法。（译者注：如果遍历完祖先链也没有找到方法的话，会调用`method_missing`方法，如果这个方法没有被定义，则抛出`NoMethodError`）当你在一个类中包含一个模块时，Ruby创建了一个匿名类来封装这个模块，并将这个匿名类插入到祖先链中，也在这个类的上方。

　　下面的代码演示了一次方法查找：

```ruby
class A
  def foo
  end
end
class B < A
  def bar
    # bar method in B
  end
end
class C < B
  def bar
    # bar method in C
    # overwriting superclass' method
  end
end

obj = C.new
obj.bar #=> in C class
obj.foo #=> not in C class
        #=>   then go to C's superclass B
        #=> not in B class
        #=>   then go to B's superclass A
        #=> execute it
```

## self

+ 在Ruby中，`self`是一个很特殊的变量，它总是指向当前对象（current object）；
+ `self`被认为是默认的receiver。也就是说，你如果没有明确指出receiver，那么系统将把`self`当做receiver；
+ 实例变量是在`self`（当前对象）中查找的。也就是说，如果我使用`@var`，那么Ruby就会在`self`所指向的对象中去寻找此实例变量。需要注意的是，实例变量并不是由类定义的，它们也和子类以及继承机制无关。

　　因此，当我们在调用一个明确指出receiver的方法时，Ruby会按照下面的步骤执行：

```ruby
obj.do_method(param)
```

+ 将`self`指向`obj`；
+ 在`self`所属的类中查找`do_method(param)`方法（方法是存放在类中，而不是实例中！）；
+ 调用方法`do_method(param)`；
