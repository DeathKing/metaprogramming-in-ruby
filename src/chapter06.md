# 块和绑定

　　每一个Ruby块中，都包含了代码以及对应的绑定。当你定义一个块的时候，它会夺过当时环境的绑定，而当你执行一个带块的方法的时候，它又会传递这个绑定。

```ruby
def who
  person = "Matz"
  yield("rocks")
end

person = "Matsumoto"

who do |y|
  puts("#{person}, #{y} the world") # => Matsumoto, rocks the world
  city = "Tokyo"
end

puts city
# => undefined local variable or method 'city' for main:Object (NameError)
```
　　观察上述代码，块附近的`person`变量在块被定义之前是一个新的局部变量，而不是`who`方法中的`person`变量。因此，块捕获了本地绑定并把它们结合在了一起。你可以在块中定义新的绑定，不过当块的生命周期结束后，它们也就消失了。

　　DeathKing注：注意变量定义的作用域。在块内部定义的变量不能用于块外部。而预先在块外部定义的变量，经过块的操作后，值会发生改变。
