8-模块
======

[编译]() <br/>
[脚本模式]() <br/>
[命名函数]() <br/>
[函数捕捉]() <br/>
[默认参数]() <br/>

Elixir中我们把许多函数组织成一个模块。我们在前几章已经提到了许多模块，如[String模块](http://elixir-lang.org/docs/stable/elixir/String.html)：
```
iex> String.length "hello"
5
```

创建自己的模块，用```defmodule```宏。用```def```宏在其中定义函数：
```
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
```
defmodule Math do
  def sum(a, b) do
    a + b
  end
end
```

这个文件可以用```elixirc```进行编译：
```
$ elixirc math.ex
```

这将生成名为```Elixir.Math.beam```的bytecode文件。
如果这时再启动iex，那么这个模块就已经可以用了（假如在含有该编译文件的目录启动iex）：
```
iex> Math.sum(1, 2)
3
```

Elixir工程通常组织在三个文件夹里：
- ebin，包括编译后的字节码
- lib，包括Elixir代码（.ex文件）
- test，测试代码（.exs文件）
<br/>

实际项目中，构建工具Mix会负责编译，并且设置好正确的路径。
而为了学习方便，Elixir也提供了脚本模式，可以更灵活而不用编译。

## 8.2-脚本模式
除了.ex文件，Elixir还支持.exs脚本文件。Elixir对两种文件一视同仁，唯一区别是.ex文件有待编译，
而.exs文件用来作脚本执行，不需要编译。例如，如下创建名为math.exs的文件：
```
defmodule Math do
  def sum(a, b) do
    a + b
  end
end

IO.puts Math.sum(1, 2)
```
执行之：
```
$ elixir math.exs
```
这中情况，文件将在内存中编译和执行，打印出“3”作为结果。没有比特码文件生成。
后文中（为了学习和练习方便），推荐使用脚本模式执行学到的代码。

## 8.3-命名函数
在某模块中，我们可以用```def/2```宏定义函数，用```defp/2```定义私有函数。
用```def/2```定义的函数可以被其它模块中的代码使用，而私有函数仅在定义它的模块内使用。
```
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
```
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
本教程中提到某函数，都是用```name/arity```的形式描述。























