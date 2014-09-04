16-协议
========
[协议和结构体]()<br/>
[回归大众化]()<br/>
[内建协议]()<br/>

协议是实现Elixir多态性的重要机制。任何数据类型只要实现了某协议，那么该协议的分发就是可用的。
让我们看个例子。

在Elixir中，只有false和nil被认为是false。其它的值都被认为是true。
根据程序需要，有时需要一个```blank?```协议，返回一个布尔值，以说明该参数是否为空。
举例来说，一个空列表或者空二进制可以被认为是空的。

我们可以如下定义协议：
```
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

这个协议期待一个函数```blank?```，它接受一个待实现的参数。
我们为不同的数据类型实现这个协议：
```
# Integers are never blank
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# Just empty list is blank
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

# Just empty map is blank
defimpl Blank, for: Map do
  # Keep in mind we could not pattern match on %{} because
  # it matches on all maps. We can however check if the size
  # is zero (and size is a fast operation).
  def blank?(map), do: map_size(map) == 0
end

# Just the atoms false and nil are blank
defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end
```

我们可以为所有内建数据类型实现协议：
  - 原子
  - BitString
  
