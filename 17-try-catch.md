17-异常处理
===========

Elixir有三种错误处理机制：errors，throws和exits。
本章我们将逐个讲解它们，包括应该在何时使用哪一个。

## Errors

错误（errors，或者叫它异常）用在代码中出现意外的地方。
举个例子，尝试让原子加上一个数字，就会返回一个错误：

```elixir
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

`raise/1`可以在任何时候激发一个运行时错误：

```elixir
iex> raise "oops"
** (RuntimeError) oops
```

也可以调用`raise/2`抛出错误，并且附上错误名称和一个键值列表：

```elixir
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

你可以定义一个模块，在里面使用`defexception/2`定义你自己的错误。
这种方式，你创建的错误和模块同名。最常见的是定义一个有详细错误信息字段的错误：

```elixir
iex> defmodule MyError do
iex>   defexception message: "default message"
iex> end
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

错误可以用`try/catch`结构 **拯救（rescued）** ：

```elixir
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "oops"}
```

这个例子处理了一个运行时异常，返回该错误本身（被显示在`:iex`回话中）。

如果你不需要使用错误对象，你可以不提供它：

```elixir
iex> try do
...>   raise "oops"
...> rescue
...>   RuntimeError -> "Error!"
...> end
"Error!"
```

在实际应用中，Elixir程序员极少使用`try/rescue`结构。
例如，当文件打开失败，很多编程语言会强制你去处理异常。
而Elixir提供的`File.read/1`函数不管文件打开成功与否，都会返回包含结果信息的元组：

```elixir
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```

这个例子中没有用到`try/rescue`。
如果你想处理打开文件可能发生的不同结果，你可以使用`case`结构来做模式匹配：

```elixir
iex> case File.read "hello" do
...>   {:ok, body}      -> IO.puts "Success: #{body}"
...>   {:error, reason} -> IO.puts "Error: #{reason}"
...> end
```

使用这种匹配处理，你就可以自己决定打开文件异常是不是一种错误。
这也是为什么Elixir不让`File.read/1`等函数自己抛出异常。
它把决定权留给程序员，让他们寻找最合适的处理方法。

如果你真的期待文件存在（_打开文件时文件不存在_ 确实是一个 **错误**），
你可以简单地使用`File.read!/1`：

```elixir
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:305: File.read!/1
```

但是标准库中的许多函数都遵循这样的模式：不返回元组来模式匹配，而是抛出异常。
具体说，这种约定是在定义名字不带感叹号后缀的函数时，返回结果信息的元组；
定义带有感叹号后缀的相同函数时，使用触发异常的机制。
可以阅读`File`模块的代码，有很多关于这种约定的好例子。

总之在Elixir中，我们避免使用`try/rescue`是因为我们**不通过错误处理机制来控制程序执行流程**。
我们视错误为其字面意思：它们只不过是用来处理意外时用到的信息。
如果你真的希望异常机制改变程序执行流程，可以使用`throws`。

## Throws

在Elixir中，你可以抛出（throw）一个值（不一定是一个错误/异常对象）做后续处理并终止当前其它操作。
（前方文字高能=>）`throw`和`catch`就被保留用来处理这样有值被抛出、但是不用`try/catch`就取不到的情况。

>译注：这句话原文也差不多。仔细读读感觉逻辑类似于“X被用来描述不用X就无法描述的东西”。

这中情况很不多，非要找得话，比如当一个库的接口没有提供合适的API时。
举个例子，`Enum`模块没有提供现成的API来做这个奇葩的事情：寻找若干数字里第一个13的倍数。
你可以利用`throw`机制，在刚找到第一个符合条件的数时，抛出这个数字，并终止执行流程：

```elixir
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

>但实际上`Enum`提供了这样的API---使用`Enum.find/2`：

>```elixir
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## Exits

Elixir代码都在各种进程中运行，彼此间相互通信。当一个进程因为自然原因死亡了（如“未处理的异常），
它会发出`exit`信号。一个进程可以通过显式地发出`exit`信号来终止：

```elixir
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```

上面的例子中，被链接的进程通过发送`exit`信号（带有参数1）而终止。
Elixir shell自动处理这个消息并把它们显示在终端上。

>打比方，在某进程中执行的Elixir代码，突遇故障难以处理，便大喊一声“要死”后死亡；
或者它主动喊出一声“要死”而死亡。是该进程发送消息（即，调用函数）。

`exit`可以被`try/catch`块捕获处理：

```elixir
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

`try/catch`已经很少使用了，用它们捕获`exit`信号就更少见了。

`exit`信号是Erlang虚拟机提供的高容错性的重要部分。
进程通常都在*监督树（supervision trees）*下运行。
监督树本身也是进程，它任务就是一直等待下面被监督进程的`exit`信号。
一旦监听到退出信号，监督策略就开始工作，发送了死亡信号的被监督进程会被重启。

就是这种监督机制使得`try/catch`和`try/rescue`代码块很少用到。
与其拯救一个错误，不如让它*快速失败*。
因为在程序发生失败后，监督树会保证这个程序恢复到一个已知的初始状态去。

## After

有时候我们有必要确保某资源在执行可能会发生错误的操作后被正确地关闭或清理。
使用`try/after`结构来做这个。
例如打开一个文件，使用`after`子句来确保它在使用后被关闭，即使发生了错误：

```elixir
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

`after`块中的代码，无论`try`代码块中是否出现错误，其都会执行。
但是，也有例外。如果一个链接进程整个终止了，它代码中的`after`块可能还没有执行。
因此，我们说`after`只是个“软”保障。
幸运的是，在Elixir语言中，打开的文件永远是链接到当前进程的。如果当前进程挂了，文件也会跟着关闭。
不管执没执行`after`块。
其它一些资源，如ETS表，套接字，端口等也是这样。

有时候你想将整个函数体用`try`结构包起来，是某些代码在某情况下会后续执行。
这时，Elixir允许你不写`try`那行：

```elixir
iex> defmodule RunAfter do
...>   def without_even_trying do
...>     raise "oops"
...>   after
...>     IO.puts "cleaning up!"
...>   end
...> end
iex> RunAfter.without_even_trying
cleaning up!
** (RuntimeError) oops
```

这时，Elixir会将整个函数体的代码用`try`块包裹，不管后面是`after`，`rescue`还是`catch`。

## 变量的作用域

定义在`try/catch/rescue/after`代码块中的变量，不会泄露到外面去。
这是因为`try`代码块有可能会失败，而这些变量此时并没有正常绑定数值：

换句话说，下面这份代码是错误的：

```elixir
iex> try do
...>   raise "fail"
...>   what_happened = :did_not_raise
...> rescue
...>   _ -> what_happened = :rescued
...> end
iex> what_happened
** (RuntimeError) undefined function: what_happened/0
```

相反，你可以保存`try`表达式的返回值：

```elixir
iex> what_happened =
...>   try do
...>     raise "fail"
...>     :did_not_raise
...>   rescue
...>     _ -> :rescued
...>   end
iex> what_happened
:rescued
```

至此我们结束了对`try/catch/rescue`的介绍。
你会发现Elixir语言中的这些概念的使用频率比其他语言小，尽管的确有时使用起来也挺顺手。
