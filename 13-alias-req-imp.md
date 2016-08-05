13-alias，require和import
=================

为了实现软件重用，Elixir提供了三种指令（`alias`，`require`和`import`），
外加一个宏命令`use`，如下：

```elixir
# 给模块起别名，让它可以用 Bar 调用而非 Foo.Bar
alias Foo.Bar, as: Bar

# 确保模块已被编译且可用（通常为了宏）
require Foo

# 从 Foo 中导入函数，使之调用时不用加`Foo`前缀
import Foo

# 执行定义在 Foo 拓展点内的代码
use Foo
```

下面我们将深入细节。记住前三个之所以称之为“指令”，
是因为它们的作用域是*词法作用域（lexicla scope）*，
而`use`是一个普通拓展点（common extension point），可以将宏展开。

## alias

指令`alias`可以为任何模块设置别名。
想象一下之前使用过的`Math`模块，它针对特殊的数学运算提供了特殊的列表（list）实现：

```elixir
defmodule Math do
  alias Math.List, as: List
end
```

现在，任何对`List`的引用将被自动变成对`Math.List`的引用。
如果还想访问原来的`List`，可以加上它的模块名前缀'Elixir'：

```elixir
List.flatten             #=> uses Math.List.flatten
Elixir.List.flatten      #=> uses List.flatten
Elixir.Math.List.flatten #=> uses Math.List.flatten
```

>注意：Elixir中定义的所有模块都被定义在Elixir命名空间内。
但为方便起见，在引用它们时，你可以省略它们的前缀‘Elixir’。

别名常被使用于定义快捷方式。实际应用中，不带`:as`选项调用`alias`会
自动将别名设置为该模块名称的最后一部分：

```elixir
alias Math.List
```

就相当于：

```elixir
alias Math.List, as: List
```

注意，`alias`是**词法作用域**。也就是说，当你在某个函数中设置别名：

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

例子中`alias`指令设置的别名只在函数`plus/2`中有效，函数`minus/2`则不受影响。

## require

Elixir提供了许多宏用于元编程（可以编写生成代码的代码）。

宏是在编译时被执行和展开的代码。
也就是说为了使用宏，你需要确保定义这个宏的模块及实现在你的代码的编译时可用（即被加载）。
这使用`require`指令实现：

```elixir
iex> Integer.odd?(3)
** (CompileError) iex:1: you must require Integer before invoking the macro Integer.odd?/1
iex> require Integer
nil
iex> Integer.odd?(3)
true
```

Elixir中，`Integer.odd?/1`函数被定义为一个宏，因此它可以被当作卫兵表达式（guards）使用。
为了调用这个宏，首先需要使用`require`引用`Integer`模块。

总的来说，一个模块在被用到之前不需要早早地require，除非我们需要用到这个模块中定义的宏的时候。
尝试调用一个没有加载的宏时，会报出一个异常。
注意，像`alias`指令一样，`require`指令也是词法作用域的。
在后面章节我们会进一步讨论宏。

## import

当想轻松地访问模块中的函数和宏时，可以使用`import`指令避免输入模块的完整名字。
例如，如果我们想多次使用`List`模块中的`duplicate/2`函数，我们可以import它：

```elixir
iex> import List, only: [duplicate: 2]
List
iex> duplicate :ok, 3
[:ok, :ok, :ok]
```

这个例子中，我们只从List模块导入了函数`duplicate`（元数是2的那个）。
尽管`:only`选项是可选的，但是仍推荐使用，以避免向当前命名空间内导入这个模块内定义的所有函数。
还有`:except`选项，可以*排除*一些函数而导入其余的。

还有选项`:only`，传递给它`:macros`或`:functions`，来导入该模块的所有宏或函数。
如下面例子，程序仅导入`Integer`模块中定义的所有的宏：

```elixir
import Integer, only: :macros
```

或者，仅导入所有的函数：

```elixir
import Integer, only: :functions
```

注意，`import`也遵循**词法作用域**，意味着我们可以在某特定函数定义内导入宏或方法：

```elixir
defmodule Math do
  def some_function do
    import List, only: [duplicate: 2]
    duplicate(:ok, 10)
  end
end
```

在这个例子中，导入的函数`List.duplicate/2`只在函数`some_function`中可见，
在该模块的其它函数中都不可用（自然，别的模块也不受影响）。

注意，若`import`一个模块，将自动`require`它。

## use

尽管不是一条指令，`use`是一个宏，与帮助你在当前上下文中使用模块的`require`指令联系紧密。
`use`宏常被用来引入外部的功能到当前的词法作用域---通常是模块。

例如，在编写测试时，我们使用ExUnit框架。开发者需要使用`ExUnit.Case` 模块：

```elixir
defmodule AssertionTest do
  use ExUnit.Case, async: true

  test "always pass" do
    assert true
  end
end
```

在代码背后，`use`宏先是`require`所给的模块，然后在模块上调用`__using__/1`回调函数，
从而允许这个模块在当前上下文中注入某些代码。

比如下面这个模块：

```exlixir
defmodule Example do
  use Feature, option: :value
end
```

会被编译成（即宏`use`扩展）

```exlixir
defmodule Example do
  require Feature
  Feature.__using__(option: :value)
end
```

到这里，关于Elixir的模块基本上讲得差不多了。后面会讲解模块的属性（Attribute）。

## 别名机制

讲到这里你会问，Elixir的别名到底是什么，它是怎么实现的？

Elixir中的别名是以大写字母开头的标识符（像`String`, `Keyword`），在编译时会被转换为原子。
例如，别名‘String’默认情况下会被转换为原子`:"Elixir.String"`：

```elixir
iex> is_atom(String)
true
iex> to_string(String)
"Elixir.String"
iex> :"Elixir.String" == String
true
```

使用`alias/2`指令，其实只是简单地改变了这个别名将要转换的结果。

别名会被转换为原子，是因为在Erlang虚拟机（以及上面的Elixir）中，模块是由原子表述。
例如，我们调用一个Erlang模块函数的机制是：

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

我们只是简单地在原子`:lists`上调用了函数`flatten`。

## 模块嵌套

我们已经介绍了别名，现在可以讲讲嵌套（nesting）以及它在Elixir中是如何工作的。
考虑下面的例子：

```elixir
defmodule Foo do
  defmodule Bar do
  end
end
```

该例子定义了两个模块：`Foo`和`Foo.Bar`。
后者在`Foo`中可以用`Bar`为名来访问，因为它们在同一个词法作用域中。
上面的代码等同于：

```elixir
defmodule Elixir.Foo do
  defmodule Elixir.Foo.Bar do
  end
  alias Elixir.Foo.Bar, as: Bar
end
```

如果之后开发者决定把`Bar`模块定义挪出`Foo`模块的定义，但是在`Foo`中仍然使用`Bar`来引用，
那它就需要以全名（Foo.Bar）来命名，或者向上文提到的，在`Foo`中设置个别名来指代。

**注意：** 在Elixir中，你并不是必须在定义`Foo.Bar`模块之前定义`Foo`模块，
因为编程语言会将所有模块名翻译成原子。
你可以定义任意嵌套的模块而不需要注意其名称链上的先后顺序
（比如，在定义`Foo.Bar.Baz`前不需要提前定义`foo`或者`Foo.Bar`）。

在后面几章我们会看到，别名在宏里面也扮演着重要角色，来保证它们是“干净”（hygienic）的。

## 一次、多个

从Elixir v1.2版本开始，可以一次性使用alias，import，require操作多个模块。
这在定义和使用嵌套模块的时候非常有用，这也是在构建Elixir程序的常见情形。

例如，假设你的程序所有模块都嵌套在`MyApp`下，
你可以一次同时给三个模块：`MyApp.Foo`,`MyApp.Bar`和`MyApp.Baz`提供别名：

```elixir
alias MyApp.{Foo, Bar, Baz}
```
