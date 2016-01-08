13-别名和代码引用
=================
[别名](#131-%E5%88%AB%E5%90%8D)    
[require](#132-require)    
[import](#133-import)    
[别名机制](#134-%E5%88%AB%E5%90%8D%E6%9C%BA%E5%88%B6)    
[嵌套](#135-%E5%B5%8C%E5%A5%97)   

为了实现软件重用，Elixir提供了三种指令（directives）。
之所以称之为“指令”，是因为它们的作用域是词法作用域（lexicla scope）。

## 13.1-别名
宏```alias```可以为任何模块名设置别名。
想象一下Math模块，它针对特殊的数学运算使用了特殊的列表实现：
```elixir
defmodule Math do
  alias Math.List, as: List
end
```   

现在，任何对```List```的引用将被自动变成对```Math.List```的引用。
如果还想访问原来的```List```，需要前缀'Elixir'：
```elixir
List.flatten             #=> uses Math.List.flatten
Elixir.List.flatten      #=> uses List.flatten
Elixir.Math.List.flatten #=> uses Math.List.flatten
```

>Elixir中定义的所有模块都在一个主Elixir命名空间。
但是为方便起见，我们平时都不再前面加‘Elixir’。

别名常被使用于定义快捷方式。实际上不带```as```选项去调用```alias```会
自动将这个别名设置为模块名的最后一部分：
```elixir
alias Math.List
```
就相当于：
```elixir
alias Math.List, as: List
```

注意，别名是**词法作用域**。即，你在某个函数中设置别名：
```elixir
defmodule Math do
  def plus(a, b) do
    alias Math.List
    # ...
  end

  def minus(a, b) do
    # ...
  end
end
```
例子中```alias```指令只在函数```plus/2```中有用，```minus/2```不受影响。

## 13.2-require
Elixir提供了许多宏，用于元编程（写能生成代码的代码）。

宏也是一堆代码，但它在编译时被展开和执行。
就是说为了使用一个宏，你需要确保它定义的模块和实现在编译期时可用。   

要使用宏，需要用```require```指令引入定义宏的模块：
```elixir
iex> Integer.odd?(3)
** (CompileError) iex:1: you must require Integer before invoking the macro Integer.odd?/1
iex> require Integer
nil
iex> Integer.odd?(3)
true
```

Elixir中，```Integer.odd?/1```函数被定义为一个宏，它可以被当作卫兵表达式（guards）使用。
为了调用这个宏，首先得用```require```引用了_Integer_模块。   

总的来说，宏在被用到之前不需要早早被require引用进来，除非我们想让这个宏在整个模块中可用。
尝试调用一个没有引入的宏会导致报错。
注意，像```alias```指令一样，```require```也是词法作用域的。
下文中我们会进一步讨论宏。

## 13.3-import
当想轻松地访问别的模块中的函数和宏时，可以使用```import```指令加上宏定义模块的完整名字。
例如，如果我们想多次使用List模块中的```duplicate```函数，我们可以这么import它：
```elixir
iex> import List, only: [duplicate: 2]
nil
iex> duplicate :ok, 3
[:ok, :ok, :ok]
```

这个例子中，我们只从List模块导入了函数```duplicate/2```。
尽管```only:```选项是可选的，但是推荐使用。   

除了```only:```选项，还有```except:```选项。
使用选项```only:```，还可以传递给它```:macros```或```:functions```。
如下面例子，程序仅导入Integer模块中所有的宏：
```elixir
import Integer, only: :macros
```
或者，仅导入所有的函数：
```elixir
import Integer, only: :functions
```

注意，```import```也是**词法作用域**，意味着我们可以在某特定函数中导入宏或方法：
```elixir
defmodule Math do
  def some_function do
    import List, only: [duplicate: 2]
    # call duplicate
  end
end
```
在此例子中，导入的函数```List.duplicate/2```只在那个函数中可见。
该模块的其它函数中都不可用（自然，别的模块也不受影响）。

注意，若import一个模块，将自动require它。

## 13.4-别名机制
讲到这里你会问，Elixir的别名到底是什么，它是怎么实现的？   

Elixir中的别名是以大写字母开头的标识符（像String, Keyword等等），在编译时会被转换为原子。
例如，别名‘String’会被转换为```:"Elixir.String"```：
```elixir
iex> is_atom(String)
true
iex> to_string(String)
"Elixir.String"
iex> :"Elixir.String"
String
```

使用```alias/2```指令，其实只是简单地改变了这个别名将要转换的结果。

别名如此这般工作，是因为在Erlang虚拟机中（以及上面的Elixir），模块名是被表述成原子的。
例如，我们调用一个Erlang模块的机制是：
```elixir
iex> :lists.flatten([1,[2],3])
[1, 2, 3]
```

这也是允许我们动态调用模块函数的机制：
```elixir
iex> mod = :lists
:lists
iex> mod.flatten([1,[2],3])
[1,2,3]
```
一句话，我们只是简单地在原子```:lists```上调用了函数```flatten```。

## 13.5-嵌套
介绍了别名，现在可以讲讲嵌套（nesting）以及它在Elixir中是如何工作的。

考虑下面的例子：
```elixir
defmodule Foo do
  defmodule Bar do
  end
end
```
该例子定义了两个模块_Foo_和_Foo.Bar_。
后者在_Foo_中可以用_Bar_为名来访问，因为它们在同一个词法作用域中。
如果之后开发者决定把Bar模块挪到另一个文件中，那它就需要以全名（Foo.Bar）或者别名来指代。

换句话说，上面的代码等同于：
```elixir
defmodule Elixir.Foo do
  defmodule Elixir.Foo.Bar do
  end
  alias Elixir.Foo.Bar, as: Bar
end
```
## 13.6-多个
From Elixir v1.2, it is possible to alias, import or require multiple modules at once. This is particularly useful once we start nesting modules, which is very common when building Elixir applications. For example, imagine you have an application where all modules are nested under MyApp, you can alias the modules MyApp.Foo, MyApp.Bar and MyApp.Baz at once as follows:

```elixir
alias MyApp.{Foo, Bar, Baz}
```
## 13.7-use
Although not a directive, use is a macro tightly related to require that allows you to use a module in the current context. The use macro is frequently used by developers to bring external functionality into the current lexical scope, often modules.

For example, in order to write tests using the ExUnit framework, a developer should use the ExUnit.Case module:
```elixir
defmodule AssertionTest do
  use ExUnit.Case, async: true

  test "always pass" do
    assert true
  end
end
```
Behind the scenes, use requires the given module and then calls the __using__/1 callback on it allowing the module to inject some code into the current context. Generally speaking, the following module:
```exlixir
defmodule Example do
  use Feature, option: :value
end
```
is compiled into
```exlixir
defmodule Example do
  require Feature
  Feature.__using__(option: :value)
end
···
With this we are almost finishing our tour about Elixir modules. The last topic to cover is module attributes.
在以后章节我们可以看到，别名在宏机制中扮演了很重要的角色，来保证宏是干净的（hygienic）。

讨论到这里，模块基本上讲得差不多了。之后会讲解模块的属性。
