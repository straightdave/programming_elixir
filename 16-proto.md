16-协议
========
[协议和结构体](#161-%E5%8D%8F%E8%AE%AE%E5%92%8C%E7%BB%93%E6%9E%84%E4%BD%93)<br/>
[回归大众](#162-%E5%9B%9E%E5%BD%92%E5%A4%A7%E4%BC%97)<br/>
[内建协议](#163-%E5%86%85%E5%BB%BA%E5%8D%8F%E8%AE%AE)<br/>

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
  - 浮点型
  - 函数
  - 整型
  - 列表
  - 图
  - PID
  - Port
  - 引用
  - 元祖

现在手边有了一个定义并被实现的协议，如此使用之：
```
iex> Blank.blank?(0)
false
iex> Blank.blank?([])
true
iex> Blank.blank?([1, 2, 3])
false
```

给它传递一个并没有实现该协议的数据类型，会导致报错：
```
iex> Blank.blank?("hello")
** (Protocol.UndefinedError) protocol Blank not implemented for "hello"
```

## 16.1-协议和结构体
协议和结构体一起使用能够大大加强Elixir的可扩展性。<br/>

在前面几章中我们知道，尽管结构体就是图，但是它们和图并不共享各自协议的实现。
像前几章一样，我们先定义一个名为```User```的结构体：
```
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
 ```
 然后看看：
 ```
iex> Blank.blank?(%{})
true
iex> Blank.blank?(%User{})
** (Protocol.UndefinedError) protocol Blank not implemented for %User{age: 27, name: "john"}
```

结构体没有使用协议针对图的实现，而是使用它自己的协议实现：
```
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

如果愿意，你可以定义你自己的语法来检查一个user为不为空。
不光如此，你还可以使用结构体创建更强健的数据类型，比如队列，然后实现所有相关的协议，比如枚举（Enumerable）或检查是否为空。

有些时候，程序员们希望给结构体提供某些默认的协议实现。因为显式给所有结构体都实现某些协议实在是太枯燥了。
这引出了下一节“回归大众”（falling back to any）的说法。

## 16.2-回归大众
能够给所有类型提供默认的协议实现肯定是很方便的。在定义协议时，把```@fallback_to_any```设置为```true```即可：
```
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```
现在这个协议可以被这么实现：
```
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

现在，那些我们还没有实现```Blank```协议的数据类型（包括结构体）也可以来判断是否为空了。

## 16.3-内建协议
Elixir自带了一些内建协议。在前面几章中我们讨论过枚举模块，它提供了许多方法。
只要任何一种数据结构它实现了Enumerable协议，就能使用这些方法：

```
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2,4,6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

另一个例子是```String.Chars```协议，它规定了如何将包含字符的数据结构转换为字符串类型。
它暴露为函数```to_string```：
```
iex> to_string :hello
"hello"
```

注意，在Elixir中，字符串插值操作调用的是```to_string```函数：
```
iex> "age: #{25}"
"age: 25"
```
上面代码能工作是因为数字类型实现了```String.Chars```协议。如果传进去的是元组就会报错：
```
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当想要打印一个比较复杂的数据结构时，可以使用```inspect```函数。该函数基于协议```Inspect```：
```
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

_Inspect_协议用来将任意数据类型转换为可读的文字表述。IEx用来打印表达式结果用的就是它：
```
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "john", age: 27}
```

记住，习惯上来说，无论何时，头顶#号被插的值，会被表现成一个不合语法的字符串。
在转换为可读的字符串时丢失了信息，因此别指望还能从该字符串取回原来的那个对象：
```
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```

Elixir中还有些其它协议，但本章就讲这几个比较常用的。下一章将讲讲Elixir中的错误捕捉以及异常。
















  
