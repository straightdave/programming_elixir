16-协议（protocols）
========

协议是实现Elixir多态性的重要机制。任何数据类型只要实现了某协议，
那么基于该协议的（函数调用）消息分发就是可用的。

>先简单解释一下上面“分发（dispatching）”的意思：对于许多编程语言，
特别是支持“duck-typing”的语言来说，对象调用方法，相当于以该对象为目的，
对其发送消息（函数/方法名），希望它支持该方法调用。
这里的“协议”二字对于熟悉ruby等具有“duck-typing”特性的语言的人来说会比较容易理解。

让我们看个例子。

在Elixir中，只有`false`和`nil`被认为是“false”的。其它的值都被认为是“true”。
根据程序需要，有时需要一个`blank?`协议（注意，我们此处称之为“协议”），
返回一个布尔值，以说明该参数是否为空。
举例来说，一个空列表或者空二进制可以被认为是空的。

我们可以如下定义协议：
```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

从上面代码的语法上看，这个协议`Blank`声明了一个函数`blank?`，接受一个参数。
我们可以为不同的数据类型实现这个协议：

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

# 只有false和nil这两个原子被认为是“空”
defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end
```

我们可以为所有内建数据类型实现协议：
  - 原子
  - 比特串
  - 浮点型
  - 函数
  - 整型
  - 列表
  - 图
  - PID
  - Port
  - 引用
  - 元组

现在手上有了一个协议的定义以及其实现，可如此使用之：

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

## 协议和结构体

Elixir的可扩展性（extensiblility）来源于将协议和结构体一同使用。

在前面几章中我们知道，尽管结构体本质上就是图（map），但是它和图并不共享各自协议的实现。
像那章一样，我们先定义一个名为`User`的结构体：

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

果然，结构体没有使用这个协议针对图的实现。
因此，对于这个结构体，需要定义它的协议实现：

```elixir
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

如果愿意，你可以定义你自己的语法来检查一个user是否为空。
不光如此，你还可以使用结构体创建更强健的数据类型（比如队列），然后实现所有相关的协议，
比如`Enumerable`，或者是`Blank`等等。

## 实现`Any`

手动给所有类型实现某些协议实现很快就会变得犹如重复性劳动般枯燥无味。
在这种情况下，Elixir给出两种选择：一是显式让我们的类型继承某些已有的实现；
二是，自动给所有类型提供实现。这两种情况，我们都需要为`Any`类型写实现代码。

### 继承

Elixir允许我们继承某些有基于`Any`类型的实现。比如，我们先为`Any`实现某个协议:

```Elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

OK我们现在有个协议通用的实现了。在定义结构体时，可以显式标注其继承了`Blank`协议的实现：

```Elixir
defmodule DeriveUser do
  @derive Blank
  defstruct name: "john", age: 27
end
```

继承的时候，Elixir会为`DeriveUser`实现`Blank`协议（基于`Blank`在`Any`上的实现）。
这种方式是可选择的：结构体只会跟它们自己显式实现或继承实现的协议一起工作。

### 退化至`Any`

`@derive`注解的一个替代物是显式地告诉协议，如果没有找到（在某个类型上得）实现的时候，
使用其`Any`的实现（如果有）。在定义协议的时候设置`@fallback_to_any`为`true`即可：

```elixir
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```

假使我们在前一小节已经完成了对`Any`的实现：

```elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

现在，所有没有实现`blank`协议的数据类型（包括结构体）都会被认为是非空的。
对比`@derive`，退化至`Any`是必然的：只有没有显式提供某个实现的实现，所有类型对于某个协议，都会有默认的行为。
这项技术提供了很大的灵活性，也支持了Elixir程序员“显式先于隐式”的编码哲学。
你可以在很多库中看到`@derive`的身影。

## 内建协议

Elixir内建了一些协议。在前面几章中我们讨论过`Enum`模块，它提供了许多方法。
只要任何一种数据结构它实现了`Enumerable`协议，就能使用这些方法：

```elixir
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2,4,6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

另一个例子是`String.Chars`协议，它规定了如何将包含字符的数据结构转换为字符串类型。
它暴露为函数```to_string```：

```elixir
iex> to_string :hello
"hello"
```

注意，在Elixir中，字符串插值操作背后就调用了```to_string```函数：

```elixir
iex> "age: #{25}"
"age: 25"
```

上面代码能工作，是因为25这个数字类型实现了`String.Chars`协议。
而如果传进去的是元组就会报错：

```elixir
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当想要打印一个比较复杂的数据结构时，可以使用`inspect`函数。该函数基于协议`Inspect`：

```elixir
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

`Inspect`协议用来将任意数据类型转换为可读的文字表述。IEx用来打印表达式结果用的就是它：

```elixir
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "john", age: 27}
```

>```inspect```是ruby中非常常用的方法。
这也能看出Elixir的作者们真是绞尽脑汁把Elixir的语法尽量往ruby上靠。

记住，被执行`inspect`函数后的结果，是头顶着`#`符号的Elixir的类型描述文本，本身并不是合法的Elixir语法。
在转换为可读的文本后，数值丢失了信息，因此别指望还能从该字符串取回原来的那个东西：

```elixir
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```

Elixir中还有些其它协议，但本章就讲这几个比较常用的。

## 协议压实（consolidation）

当使用Mix构建工具的时候，你可能会看到如下输出：

```
Consolidated String.Chars
Consolidated Collectable
Consolidated List.Chars
Consolidated IEx.Info
Consolidated Enumerable
Consolidated Inspect
```

这些都是Elixir内建的协议，它们正在被“压实”（压紧、夯实；还有更好的翻译么？）。
因为协议可以被分发给所有的数据类型，在每一次调用时，协议必须检查是否对某数据类型提供了实现。
这消耗大量资源。

但是，如果使用构建工具Mix，我们会知道所有的模块已被定义，包括协议和实现。
这样，协议可以被优化到一个简单的易于快速分发的模块中。

从Elixir v1.2开始，这种优化是自动进行的。后面的《Mix和OTP教程》中会讲到。
