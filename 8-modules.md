8-模块
======

[编译](#81-%E7%BC%96%E8%AF%91)   
[脚本模式](#82-%E8%84%9A%E6%9C%AC%E6%A8%A1%E5%BC%8F)    
[命名函数](#83-%E5%91%BD%E5%90%8D%E5%87%BD%E6%95%B0)    
[函数捕捉](#84-%E5%87%BD%E6%95%B0%E6%8D%95%E6%8D%89)    
[默认参数](#85-%E9%BB%98%E8%AE%A4%E5%8F%82%E6%95%B0)    

Elixir中我们把许多函数组织成一个模块。我们在前几章已经提到了许多模块，
如[String模块](http://elixir-lang.org/docs/stable/elixir/String.html)：
```elixir
iex> String.length "hello"
5
```

创建自己的模块，用```defmodule```宏。用```def```宏在其中定义函数：
```elixir
iex> defmodule Math do
...>   def sum(a, b) do
...>     a + b
...>   end
...> end

iex> Math.sum(1, 2)
3
```
>像ruby一样，模块名大写起头

## 8.1-编译
通常把模块写进文件，这样可以编译和重用。假如文件```math.ex```有如下内容：
```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end
```

这个文件可以用```elixirc```进行编译：
```elixir
$ elixirc math.ex
```

这将生成名为```Elixir.Math.beam```的bytecode文件。
如果这时再启动iex，那么这个模块就已经可以用了（假如在含有该编译文件的目录启动iex）：
```elixir
iex> Math.sum(1, 2)
3
```

Elixir工程通常组织在三个文件夹里：
- ebin，包括编译后的字节码
- lib，包括Elixir代码（.ex文件）
- test，测试代码（.exs文件）   

实际项目中，构建工具Mix会负责编译，并且设置好正确的路径。
而为了学习方便，Elixir也提供了脚本模式，可以更灵活而不用编译。

## 8.2-脚本模式
除了.ex文件，Elixir还支持.exs脚本文件。
Elixir对两种文件一视同仁，唯一区别是.ex文件会保留编译执行后产出的比特码文件，
而.exs文件用来作脚本执行，不会留下比特码文件。例如，如下创建名为math.exs的文件：
```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end

IO.puts Math.sum(1, 2)
```

执行之：   
```sh
$ elixir math.exs
```
像这样执行脚本文件时，将在内存中编译和执行，打印出“3”作为结果。没有比特码文件生成。
后文中（为了学习和练习方便），推荐使用脚本模式执行学到的代码。

## 8.3-命名函数
在某模块中，我们可以用```def/2```宏定义函数，用```defp/2```定义私有函数。
用```def/2```定义的函数可以被其它模块中的代码使用，而私有函数仅在定义它的模块内使用。
```elixir
defmodule Math do
  def sum(a, b) do
    do_sum(a, b)
  end

  defp do_sum(a, b) do
    a + b
  end
end

Math.sum(1, 2)    #=> 3
Math.do_sum(1, 2) #=> ** (UndefinedFunctionError)
```

函数声明也支持使用卫兵或多个子句。
如果一个函数有好多子句，Elixir会匹配每一个子句直到找到一个匹配的。
下面例子检查参数是否是数字：
```elixir
defmodule Math do
  def zero?(0) do
    true
  end

  def zero?(x) when is_number(x) do
    false
  end
end

Math.zero?(0)  #=> true
Math.zero?(1)  #=> false

Math.zero?([1,2,3])
#=> ** (FunctionClauseError)
```
如果没有一个子句能匹配参数，会报错。

## 8.4-函数捕捉
本教程中提到函数，都是用```name/arity```的形式描述。
这种表示方法可以被用来获取一个命名函数（赋给一个函数型变量）。
下面用iex执行一下上文定义的math.exs文件：
```elixir
$ iex math.exs
```

```elixir
iex> Math.zero?(0)
true
iex> fun = &Math.zero?/1
&Math.zero?/1
iex> is_function fun
true
iex> fun.(0)
true
```
用```&<function notation>```通过函数名捕捉一个函数，它本身代表该函数值（函数类型的值）。
它可以不必赋给一个变量，直接用括号来使用该函数。  

本地定义的，或者已导入的函数，比如```is_function/1```，可以不用前缀模模块名：
```elixir
iex> &is_function/1
&:erlang.is_function/1
iex> (&is_function/1).(fun)
true
```

这种语法还可以作为快捷方式来创建和使用函数：
```elixir
iex> fun = &(&1 + 1)
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> fun.(1)
2
```

代码中```&1``` 表示传给该函数的第一个参数。
因此，```&(&1+1)```其实等同于```fn x->x+1 end```。在创建短小函数时，这个很方便。
想要了解更多关于```&```捕捉操作符，参考[Kernel.SpecialForms文档](http://elixir-lang.org/docs/stable/elixir/Kernel.SpecialForms.html)。

## 8.5-默认参数
Elixir中，命名函数也支持默认参数：
```elixir
defmodule Concat do
  def join(a, b, sep \\ " ") do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

任何表达式都可以作为默认参数，但是只在函数调用时 **用到了** 才被执行。
（函数定义时，那些表达式只是存在那儿，不执行；函数调用时，没有用到默认值，也不执行）。
```elixir
defmodule DefaultTest do
  def dowork(x \\ IO.puts "hello") do
    x
  end
end
```

```elixir
iex> DefaultTest.dowork 123
123
iex> DefaultTest.dowork
hello
:ok
```

如果有默认参数值的函数有了多条子句，推荐先定义一个函数头（无具体函数体）声明默认参数：
```elixir
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ")

  def join(a, b, _sep) when is_nil(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
IO.puts Concat.join("Hello")               #=> Hello
```

使用默认值时，注意对函数重载会有一定影响。考虑下面例子：
```elixir
defmodule Concat do
  def join(a, b) do
    IO.puts "***First join"
    a <> b
  end

  def join(a, b, sep \\ " ") do
    IO.puts "***Second join"
    a <> sep <> b
  end
end
```

如果将以上代码保存在文件“concat.ex”中并编译，Elixir会报出以下警告：
```elixir
concat.ex:7: this clause cannot match because a previous clause at line 2 always matches
```

编译器是在警告我们，在使用两个参数调用```join```函数时，总使用第一个函数定义。
只有使用三个参数调用时，才会使用第二个定义：

```elixir
$ iex concat.exs
```

```elixir
iex> Concat.join "Hello", "world"
***First join
"Helloworld"
iex> Concat.join "Hello", "world", "_"
***Second join
"Hello_world"
```

后面几章将介绍使用命名函数来做循环，如何从别的模块中导入函数，以及模块的属性等。
