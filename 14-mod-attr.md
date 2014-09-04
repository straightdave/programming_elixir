14-模块属性
===========
[作为注释]() <br/>
[作为常量]() <br/>
[作为临时存储]() <br/>

在Elixir中，模块属性（attributes）主要服务于三个目的：
  1. 作为一个模块的注释，通常附加上用户或虚拟机用到的信息
  2. 作为常量
  3. 在编译时作为一个临时的模块存储

让我们一个一个讲解。

## 14.1-作为注释
Elixir从Erlang带来了模块属性的概念。例子：
```
defmodule MyServer do
  @vsn 2
end
```
这个例子中，我们显式地为该模块设置了版本这个属性。
属性标识```@vsn```会被Erlang虚拟机的代码装载机制用到，读取并检查该模块是否在某处被更新了。
如果不注明版本号，会设置为这个模块函数的md5 checksum。

Elixir有个好多系统保留属性。这里只列举一些最常用的：
  - @moduledoc
    为当前模块提供文档
  - @doc
    为该属性后面的函数或宏提供文档
  - @behaviour
    （注意这个单词是英式拼法）用来注明一个OTP或用户自定义行为
  - @before_compile
    提供一个每当模块被编译之前执行的钩子。这使得我们可以在模块被编译之前往里面注入函数。

@moduledoc和@doc是很常用的属性，推荐经常使用（写文档）。

Elixir视文档为一等公民，提供了很多方法来访问文档。

让我们回到上几章定义的Math模块，为它添加文档，然后依然保存在math.ex文件中：
```
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

上面例子使用了heredocs注释。heredocs是多行的文本，用三个引号包裹，保持里面内容的格式。
下面例子演示在iex中，用h命令读取模块的注释：
```
$ elixirc math.ex
$ iex
iex> h Math # Access the docs for the module Math
...
iex> h Math.sum # Access the docs for the sum function
...
```

Elixir还提供了[ExDoc工具](https://github.com/elixir-lang/ex_doc)，利用注释文档生成HTNL页。

你可以看看[模块](http://elixir-lang.org/docs/stable/elixir/Module.html)里面列出的模块属性列表，
看看Elixir还支持那些模块属性。

Elixir还是用这些属性来定义[typespecs](http://elixir-lang.org/docs/stable/elixir/Kernel.Typespec.html)：
  - @spec
    为一个函数提供specification
  - @callback
    为行为回调函数提供spec
  - @type
    定义一个@spec中用到的类型
  - @typep
    定义一个私有类型，用于@spec
  - @opaque
    定义一个opaque类型用于@spec

本节讲了一些内置的属性。当然，属性可以被开发者、或者被类库扩展来支持自定义的行为。

## 14.2-作为常量
Elixir开发者经常会模块属性当作常量使用：
```
defmodule MyServer do
  @initial_state %{host: "147.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```

>不同于Erlang，默认情况下用户定义的属性不被存储在模块里。属性值仅在编译时存在。
开发者可以通过调用```Module.register_attribute/3```来使属性的行为更接近Erlang。

访问一个未定义的属性会报警告：
```
defmodule MyServer do
  @unknown
end
warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it to nil before access
```

最后，属性也可以在函数中被读取：
```
defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
```

注意在函数内读取某属性读取的是该属性当前值的快照。换句话说，读取的是编译时的值，而非运行时。
我们即将看到，这点使得属性可以作为模块在编译时的临时存储。

## 14.3-作为临时存储
Elixir组织中有一个项目，叫做[Plug](https://github.com/elixir-lang/plug)。这个项目的目标是创建一个Web的库和框架。

>类似于rack？

Plug库允许开发者定义它们自己的插件



























