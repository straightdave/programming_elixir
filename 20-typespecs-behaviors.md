20-Typespecs和behaviors
=======================

## 类型（type）和规格说明（spec）

Elixir是一门动态类型语言，Elixir中所有数据类型都是在运行时动态推定的。
然而，Elixir还提供了 **typespecs** 标记，用来：   

  1. 声明自定义数据类型
  2. 声明含有显式类型说明的函数签名（即函数的规格说明）

### 函数的规格说明（spec）

默认地，Elixir提供了一些基础数据类型，表示为 `integer` 或者 `pid`。
还有一些复杂情形：如函数`round/1`为例，它对一个float类型的数值四舍五入。
它以一个`number`（一个`integer`或`float`）作为参数，返回一个`integer`。

那么，它在[round函数的文档](http://elixir-lang.org/docs/stable/elixir/Kernel.html#round/1)
里面记载的函数签名为：

```
round(number) :: integer
```

`::` 表示其左边的函数 *返回* 一个其右面声明的类型的值。函数名后面括号中是参数类型的列表。

如果想特别注明某个函数的参数类型及返回值类型，那么可以在定义函数的时候，
在函数前面使用`@spec`指令附加上函数的规格说明（spec）。

比如，在函数库源码中，函数`round/1`是这么写的：

```elixir
@spec round(number) :: integer
def round(number), do: # 具体实现 ...
```

Elixir还支持组合类型。例如，整数的列表，它的类型表示为`[integer]`。
可以阅读[typespecs的文档](http://elixir-lang.org/docs/stable/elixir/typespecs.html)
查看Elixir提供的所有内建类型的表示方法。

### 定义自定义类型

Elixir提供了许多有用的内建类型，而且也方便创建自定义类型应用于特定场景。
方法是在定义的时候，加上`@type`指令。

比如我们有个模块叫做`LuosyCalculator`，可以执行常见的算术计算（如求和、计算乘积等）。
但是，它的函数不是返回结果数值，而是返回一个元組，
该元組第一个元素是计算结果，第二个是随机的文字记号。

```elixir
defmodule LousyCalculator do
  @spec add(number, number) :: {number, String.t}
  def add(x, y), do: { x + y, "你用计算器算这个？！" }

  @spec multiply(number, number) :: {number, String.t}
  def multiply(x, y), do: { x * y, "老天，不是吧？！" }
end
```

从例子中可以看出，元组是复合类型。每个元组都定义了其具体元素类型。
至于为何是`String.t`而不是`string`的原因，可以参考
[这篇文章](http://elixir-lang.org/docs/stable/elixir/typespecs.html#Notes)，
此处不多说明。

像这样定义函数规格说明是没问题，但是一次次重复写这种复合类型的
表示方法`{number, String.t}`，很快会厌烦的吧。
我们可以使用`@type`指令来声明我们自定义的类型：

```elixir
defmodule LousyCalculator do
  @typedoc """
  Just a number followed by a string.
  """
  @type number_with_remark :: {number, String.t}

  @spec add(number, number) :: number_with_remark
  def add(x, y), do: {x + y, "You need a calculator to do that?"}

  @spec multiply(number, number) :: number_with_remark
  def multiply(x, y), do: {x * y, "It is like addition on steroids."}
end
```

指令`@typedoc`，和`@doc`或`@moduledoc`指令类似，用来解释说明自定义的类型，放在`@type`前面。

另外，通过`@type`定义的自定义类型，实际上也是模块的成员，可以被外界访问：

```elixir
defmodule QuietCalculator do
  @spec add(number, number) :: number
  def add(x, y), do: make_quiet(LousyCalculator.add(x, y))

  @spec make_quiet(LousyCalculator.number_with_remark) :: number
  defp make_quiet({num, _remark}), do: num
end
```

如果想要将某个自定义类型保持私有，可以使用 `@typep` 指令代替 `@type` 。

### 静态代码分析

给函数等元素标记类型或者签名的作用，不仅仅是被用来作为程序文档说明。举个例子，
Erlang工具[Dialyzer][Dialyzer](http://www.erlang.org/doc/man/dialyzer.html)
通过这些类型或者签名标记，进行代码静态分析。
这就是为什么我们在 `QuiteCalculator` 例子中，
即使 `make_quite/1` 是个私有函数，也写了函数规格说明。

## 行为（behavior）

许多模块公用相同的公共API。可以参考下[Plug](https://github.com/elixir-lang/plug)，
正如它的描述所言，是一个用于互联网应用的、可编辑的模块的**规格声明**。
每个所谓*plug*就是一个**必须**实现至少两个公共函数：`init/1`和`call/2`的模块。

行为提供了一种方法，用来：

* 定义一系列必须实现的函数
* 确保模块实现所有这些函数

你也可以把这些行为想象为面向对象语言里的接口：模块必须实现的一系列函数签名。

### 定义行为（Defining behaviors）

假如说我们希望实现一系列parser，每个parser解析结构化的数据：比如，一个JSON parser或是YAML parser。
这两个parser的*行为*几近相同：
它们都提供一个`parse/1`函数和一个`extensions/0`函数。`parse/1`函数返回一个数据对应的Elixir表达。
而`extensions/0`函数返回一个可被其解析的文件的扩展名列表（如，JSON文件是`.json`）。

我们可以创建一个名为`Parser`的行为：

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end
```

那么，采用`Parser`这个行为的模块，必须实现所有被`@callback`指令标记的函数。正如你所看到的，
`@callback`指令不但可以接受一个函数名，还可以接受一个函数规格定义（我们在本文开头讲述的，函数的spec）。


### 采用行为（adopting behavior）

模块采用一个行为的语法非常直白：

```elixir
defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```

```elixir
defmodule YAMLParser do
  @behaviour Parser

  def parse(str), do: # ... parse YAML
  def extensions, do: ["yml"]
end
```

如果一个模块采用了一个尚未完全实现其所需回调方法的**行为（behavior）**，这将生成一个编译时错误。
