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


























