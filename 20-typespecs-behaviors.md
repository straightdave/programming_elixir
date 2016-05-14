20-Typespecs和behaviors
=======================

## 类型（type）和规格说明（spec）

Elixir是一门动态类型语言，Elixir中所有数据类型都是在运行时动态推定的。
然而，Elixir还提供了 **typespecs** 标记，用来：   

  1. 声明自定义数据类型     
  2. 声明含有显式类型说明的函数签名（即函数的规格说明）     

### 函数的规格说明（spec）

默认地，Elixir提供了一些基础数据类型，比如 `integer` 或者 `pid`。还有其它一些复杂的：
如函数`round/1`，它对一个float类型的数值四舍五入。
它以一个`number`（一个`integer`或`float`）作为参数，返回一个`integer`。

[round函数的文档](http://elixir-lang.org/docs/stable/elixir/Kernel.html#round/1)
里面描述`round/1`的函数签名为：

```
round(number) :: integer
```

`::` 表示其左边的函数 *返回* 一个其右面声明的类型的值。
函数的规格说明写在函数定义的前面、以`@spec`指令打头。
比如`round/1`函数可以这么写：

```elixir
@spec round(number) :: integer
def round(number), do: # 具体实现 ...
```

Elixir还支持组合类型。例如，整数的列表，它的类型表示为`[integer]`。
可以阅读[typespecs的文档](http://elixir-lang.org/docs/stable/elixir/typespecs.html)
查看Elixir提供的所有内建类型。

### 定义自定义类型

Elixir提供了许多有用的内建类型，而且也很方便去创建自定义类型应用于合适的场景。
在定义模块的时候，通过加上`@type`指令就可以实现。

比方我们有个模块叫做`LuosyCalculator`，可以执行常见的算术计算（如求和、计算乘积等）。
但是，它不是返回数值，而是返回一个元祖，该元祖第一个元素是计算结果，第二个是随机的文字记号。

```elixir
defmodule LousyCalculator do
  @spec add(number, number) :: {number, String.t}
  def add(x, y), do: {x + y, "你用计算器算这个？！"}

  @spec multiply(number, number) :: {number, String.t}
  def multiply(x, y), do: {x * y, "老天，不是吧？！"}
end
```

从例子中可以看出，元组是复合类型。每个元组都定义了其具体元素类型。
至于为何是`String.t`而不是`string`，可以参考
[这篇文章](http://elixir-lang.org/docs/stable/elixir/typespecs.html#Notes)。

像这样定义函数规格说明是没问题，但是一次次重复写这种复合类型的
样式名称`{number, String.t}`，很快会厌烦。
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

指令`@typedoc`，和`@doc`或`@moduledoc`指令类似，被用来记录自定义类型。

自定义类型通过`@type`定义，可以从其定义的模块导出并被外界访问：

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

Typespecs的作用不仅仅是被用来作为程序文档说明。举个例子，
Erlang工具[Dialyzer][Dialyzer](http://www.erlang.org/doc/man/dialyzer.html)
使用typespecs来进行代码静态分析。
这就是为什么我们在 `QuiteCalculator` 例子中，
即使 `make_quite/1` 是个私有函数，也写了函数规格说明。

## Behaviours

Many modules share the same public API. Take a look at [Plug](https://github.com/elixir-lang/plug), which, as its description states, is a **specification** for composable modules in web applications. Each *plug* is a module which **has to** implement at least two public functions: `init/1` and `call/2`.

Behaviours provide a way to:

* define a set of functions that have to be implemented by a module;
* ensure that a module implements all the functions in that set.

If you have to, you can think of behaviours like interfaces in object oriented languages like Java: a set of function signatures that a module has to implement.

### Defining behaviours

Say we want to implement a bunch of parsers, each parsing structured data: for example, a JSON parser and a YAML parser. Each of these two parsers will *behave* the same way: both will provide a `parse/1` function and an `extensions/0` function. The `parse/1` function will return an Elixir representation of the structured data, while the `extensions/0` function will return a list of file extensions that can be used for each type of data (e.g., `.json` for JSON files).

We can create a `Parser` behaviour:

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end
```

Modules adopting the `Parser` behaviour will have to implement all the functions defined with the `@callback` directive. As you can see, `@callback` expects a function name but also a function specification like the ones used with the `@spec` directive we saw above.

### Adopting behaviours

Adopting a behaviour is straightforward:

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


If a module adopting a given behaviour doesn't implement one of the callbacks required by that behaviour, a compile-time warning will be generated.
