14-模块属性
===========

在Elixir中，模块属性（module attributes）主要服务于三个目的：
  1. 作为一个模块的注解（annotations），通常附加上用户或虚拟机会用到的信息
  2. 作为常量
  3. 在编译时作为一个临时的模块存储机制

下面让我们来一一讲解。

## 作为注解（annotations）

Elixir从Erlang带来了模块属性的概念。如：

```elixir
defmodule MyServer do
  @vsn 2
end
```

这个例子中，我们显式地为该模块设置了 _版本(vsn即version)_ 属性。
`@vsn`是一个系统保留的属性名称，它被Erlang虚拟机的代码装载机制使用，以检查该模块是否被更新过。
如果不注明版本号，该属性的值会自动设置为模块函数的md5 checksum。

Elixir还有好多系统保留的预定义注解。下面是一些比较常用的：

  * `@moduledoc` - 为当前模块提供文档说明
  * `@doc` - 为该属性标注的函数或宏提供文档说明
  * `@behaviour` - （注意这个单词是英式拼法）用来注明一个OTP或用户自定义行为
  * `@before_compile` - 提供一个每当模块被编译之前执行的钩子。这使得我们可以在模块被编译之前往里面注入函数

`@moduledoc`和`@doc`是目前最常用的注解属性，我们也希望你能够多使用它们。
Elixir视文档为一等公民，而且提供了很多方法来访问这些文档。
你可以拓展阅读文章[《用我们官方的方式写Elixir程序文档》](http://elixir-lang.org/docs/stable/elixir/writing-documentation.html)。

让我们回到上几章定义的`Math`模块，为它添加文档，然后依然保存在math.ex文件中：

```elixir
defmodule Math do
  @moduledoc """
  Provides math-related functions.

  ## Examples

      iex> Math.sum(1, 2)
      3

  """

  @doc """
  Calculates the sum of two numbers.
  """
  def sum(a, b), do: a + b
end
```

Elixir推荐使用markdown语法和多行文本（heredocs）书写容易阅读的文档。
heredocs是多行的字符串，用三个双引号包裹，它会保持里面内容的格式不变。
我们可以在IEx中读取任何编译的模块的文档：

```elixir
$ elixirc math.ex
$ iex
```

```
iex> h Math # Access the docs for the module Math
...
iex> h Math.sum # Access the docs for the sum function
...
```

Elixir还提供了[ExDoc工具](https://github.com/elixir-lang/ex_doc)，
利用注释生成HTML页文档。

你可以看看[模块](http://elixir-lang.org/docs/stable/elixir/Module.html)
里面列出的完整的模块注解列表，Elixir还利用注解来定义[typespecs](../20-typespecs-behaviors.md)。

本节讲了一些内置的注解。当然，注解可以被开发者和类库扩展使用，来支持自定义的行为。

## 作为常量

Elixir开发者经常会将模块属性当作常量使用：

```elixir
defmodule MyServer do
  @initial_state %{host: "147.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```

>不同于Erlang，默认情况下用户定义的属性不会被存储在模块里。属性值仅在编译时存在。
开发者可以通过调用`Module.register_attribute/3`来使这种属性的行为更接近Erlang。

访问一个未定义的属性会报警告：

```elixir
defmodule MyServer do
  @unknown
end
warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it to nil before access
```

最后，属性也可以在函数中被读取：

```elixir
defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
```

注意，在函数内读取某属性，读取的是该属性值的一份快照。换句话说，读取的是编译时的值，而非运行时。
后面我们将看到，这个特点使得属性可以作为模块在编译时的临时存储，十分有用。

## 作为临时存储

Elixir组织中有一个项目，叫做[Plug](https://github.com/elixir-lang/plug)，
这个项目的目标是创建一个通用的Web库和框架。

>注：我想功能应该类似于ruby的rack。你可以定义各种plug，这这些plug会像链条一样，
按顺序各自对http请求进行加工处理，最后返回。这类似给rails或sinatra定义各种rack中间件，
也有些类似Java filter的概念。最终，Plug框架会组织和执行它们。

Plug库允许开发者定义它们自己的plug，运行在web服务器上：

```elixir
defmodule MyPlug do
  use Plug.Builder

  plug :set_header
  plug :send_ok

  def set_header(conn, _opts) do
    put_resp_header(conn, "x-header", "set")
  end

  def send_ok(conn, _opts) do
    send(conn, 200, "ok")
  end
end

IO.puts "Running MyPlug with Cowboy on http://localhost:4000"
Plug.Adapters.Cowboy.http MyPlug, []
```

上面例子中，我们用了`plug/1`宏来连接处理请求时会被调用的函数。
在代码背后，每次调用宏`plug/1`时，Plug库把提供的参数（即plug的名字）存储在`@plugs`属性里。
就在模块被编译之前，Plug会执行一个回调函数，该回调函数定义一个函数（`call/2`）来处理http请求。
这个函数将按顺序执行所有保存在`@plugs`属性里的plugs。

要理解底层的代码，我们还需要了解宏，因此我们将在后期《元编程》章节中回顾这个模式。
这里的重点是怎样使用属性来存储数据，让开发者可以创建DSL（领域特定语言）。

另一个例子来自[ExUnit框架](http://elixir-lang.org/docs/stable/ex_unit/)，
它使用模块属性作为注解和存储：

```elixir
defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
```

ExUnit中，标签（Tag）被用来注解该测试用例。在标记之后，这些标签可以用来过滤测试用例。
例如，你可以避免执行那些被标记成`:external`的测试，因为它们执行起来很慢而且可以依赖外部的东西。
但是它们依然在你的工程之内。

本章带你一窥Elixir元编程的冰山一角，讲解了模块属性在开发中是如何扮演关键角色的。
下一章将讲解结构体（structs）和协议（protocols），在前进到其它更远的知识点（诸如异常处理等）之前。
