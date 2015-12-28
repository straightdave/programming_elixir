16-协议
========
[协议和结构体](#161-%E5%8D%8F%E8%AE%AE%E5%92%8C%E7%BB%93%E6%9E%84%E4%BD%93)    
[回归一般化](#)    
[内建协议](#163-%E5%86%85%E5%BB%BA%E5%8D%8F%E8%AE%AE)    

协议是实现Elixir多态性的重要机制。任何数据类型只要实现了某协议，那么该协议的分发就是可用的。
让我们看个例子。

>这里的“协议”二字对于熟悉ruby等具有duck-typing特性的语言的人来说会比较容易理解。

在Elixir中，只有false和nil被认为是false的。其它的值都被认为是true。
根据程序需要，有时需要一个```blank?```协议（注意，我们此处称之为“协议”），
返回一个布尔值，以说明该参数是否为空。
举例来说，一个空列表或者空二进制可以被认为是空的。

我们可以如下定义协议：
```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

从上面代码的语法上看，这个协议```Blank```声明了一个函数```blank?```，接受一个参数。
看起来这个“协议”像是一份声明，需要后续的实现。
下面我们为不同的数据类型实现这个协议：
```elixir
# 整型永远不为空
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# 只有空列表是“空”的
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

# 只有空map是“空”
defimpl Blank, for: Map do
  # 一定要记住，我们不能匹配 %{} ，因为它能match所有的map。
  # 但是我们能检查它的size是不是0
  # 检查size是很快速的操作
  def blank?(map), do: map_size(map) == 0
end

# 只有false和nil这两个原子被认为是空得
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
```elixir
iex> Blank.blank?(0)
false
iex> Blank.blank?([])
true
iex> Blank.blank?([1, 2, 3])
false
```

给它传递一个并没有实现该协议的数据类型，会导致报错：
```elixir
iex> Blank.blank?("hello")
** (Protocol.UndefinedError) protocol Blank not implemented for "hello"
```

## 16.1-协议和结构体
协议和结构体一起使用能够加强Elixir的可扩展性。    

在前面几章中我们知道，尽管结构体本质上就是图（map），但是它们和图并不共享各自协议的实现。
像前几章一样，我们先定义一个名为```User```的结构体：
```elixir
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end
{:module, User, <<70, 79, 82, ...>>, {:__struct__, 0}}
```
然后看看能不能用刚才定义的协议：
```elixir
iex> Blank.blank?(%{})
true
iex> Blank.blank?(%User{})
** (Protocol.UndefinedError) protocol Blank not implemented for %User{age: 27, name: "john"}
```

果然，结构体没有使用协议针对图的实现。
因此，结构体需要使用它自己的协议实现：
```elixir
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

如果愿意，你可以定义你自己的语法来检查一个user是否为空。
不光如此，你还可以使用结构体创建更强健的数据类型（比如队列），然后实现所有相关的协议
（就像枚举```Enumerable```那样），检查是否为空等等。

有些时候，程序员们希望给结构体提供某些默认的协议实现，因为显式给所有结构体都实现某些协议实在是太枯燥了。
这引出了下一节“回归一般化”（falling back to any）的说法。

## 16.2-回归一般化
能够给所有类型提供默认的协议实现肯定是很方便的。
在定义协议时，把```@fallback_to_any```设置为```true```即可：
```elixir
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```
现在这个协议可以被这么实现：
```elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

现在，那些我们还没有实现```Blank```协议的数据类型（包括结构体）也可以来判断是否为空了
（虽然默认会被认为是false，哈哈）。

## 16.3-内建协议
Elixir自带了一些内建的协议。在前面几章中我们讨论过枚举模块，它提供了许多方法。
只要任何一种数据结构它实现了Enumerable协议，就能使用这些方法：

```elixir
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2,4,6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

另一个例子是```String.Chars```协议，它规定了如何将包含字符的数据结构转换为字符串类型。
它暴露为函数```to_string```：
```elixir
iex> to_string :hello
"hello"
```

注意，在Elixir中，字符串插值操作里面调用了```to_string```函数：
```elixir
iex> "age: #{25}"
"age: 25"
```
上面代码能工作，是因为25是数字类型，而数字类型实现了```String.Chars```协议。
如果传进去的是元组就会报错：
```elixir
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当想要打印一个比较复杂的数据结构时，可以使用```inspect```函数。该函数基于协议```Inspect```：
```elixir
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

_Inspect_ 协议用来将任意数据类型转换为可读的文字表述。IEx用来打印表达式结果用的就是它：
```elixir
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "john", age: 27}
```

>```inspect```是ruby中非常常用的方法。
这也能看出Elixir的作者们真是绞尽脑汁把Elixir的语法尽量往ruby上靠。

记住，头顶着#号被插的值，会被```to_string```表现成纯字符串。
在转换为可读的字符串时丢失了信息，因此别指望还能从该字符串取回原来的那个对象：
```elixir
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```

Elixir中还有些其它协议，但本章就讲这几个比较常用的。下一章将讲讲Elixir中的错误捕捉以及异常。
