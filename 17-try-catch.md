17-异常处理
===========
[Errors]()<br/>
[Throws]()<br/>
[Exits]()<br/>
[After]()<br/>
[变量作用域]()<br/>

Elixir有三种错误机制：errors，throws和exits。本章我们将逐个讲解它们，包括在何时使用哪一个。

## 17.1-Errors
举个例子，尝试让原子加上一个数字，就会激发一个错误（errors）：
```
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

使用宏```raise/1```可以在任何时候激发一个运行时错误：
```
iex> raise "oops"
** (RuntimeError) oops
```

用```raise/2```，并且附上错误名称和一个键值列表可以激发规定好的错误：
```
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

你好可以使用```defexception/2```定义你自己的错误。最常见的是定义一个有消息说明的错误：
```
iex> defexception MyError, message: "default message"
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

用```try/catch```结构可以处理异常：
```
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
RuntimeError[message: "oops"]
```
这个例子处理了一个运行时异常，返回该错误本身（会被显示在IEx对话中）。
在实际操作中，Elixir程序员很少使用```try/rescue```结构。
例如，当文件打开失败，很多编程语言会强制你去处理一个异常。而Elixir提供的```File.read/1```函数返回包含信息的元组，不管文件打开成功与否：
```
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```
这个例子中没有```try/rescue```。如果你想处理打开文件可能的不同结果，你可以使用case来匹配：
```
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```
使用这个匹配处理，你可以自己决定要不要把问题抛出来。
这就是为什么Elixir不让```File.read/1```等函数自己抛出异常。它把决定权留给程序员，让他们寻找最合适的处理方法。


如果你真的期待文件存在（打开文件时文件不存在确实是一个错误），你可以简单地使用```File.read!/1```：
```
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:305: File.read!/1
```

换句话说，我们避免使用```try/rescue```是因为我们**不用错误处理来控制程序执行流程**。
在Elixir中，我们视错误为其字面意思：它们之不说是用来表示意外或异常信息。
如果你真的希望改变执行过程，你可以使用```throws```。

## 17.2-Throws
在Elixir中，你可以抛出（throw）一个值稍后处理。```throw```和```catch```就被保留着为了处理一些你抛出了值，但是不用```try/catch```就取不到的情况。

这些情况实际中很少出现，除非当一个库的接口没有提供合适的API等情况。
例如，假如枚举模块没有提供任何API来寻找某范围内第一个13的倍数：
```
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

但是它提供了这样的函数```Enum.find/2```：
```
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## 17.3-Exits
每段Elixir代码都在进程中运行。进程与进程小虎交流。当一个进程终止了，它会发出```exit```信号。
一个进程可以通过显式地发出这个信号来终止：
```
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```
上面的例子中，链接着的进程通过发送```exit```信号（带有参数数字1）而终止。Elixir shell自动处理这个信息并把它们显示在终端上。

exit还可以被```try/catch```块捕获处理：
```
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

使用```try/catch```已经很少用了，用它们捕获exit信号就更少见了。

exit信号是Erlang虚拟机提供的高容错性的重要部分。进程通常都在*监督树（supervision trees）*下运行。
监督树本身也是进程，它们通过exit信号监督其它进程。然后通过某些策略决定是否重启。

就是这种监督系统使得```try/catch```和```try/rescue```代码块很少用到。预期处理一个错误，不如让它*快速失败*。
因为在失败后，监督树会保证我们的程序将恢复到一个已知的初始状态去。

## 17.4-After
有时候有必要使用```try/after```来保证某资源在使用后被正确关闭或清除。
例如，我们打开一个文件，然后使用```try/after```来确保它在使用后被关闭：
```
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

## 17.5-变量作用域
对于定义在```try/catch/rescue/after```代码块中的变量，切记不可让它们泄露到外面去。这时因为```try```代码块有可能会失败，而这些变量此时并没有正常绑定数值：
```
iex> try do
...>   from_try = true
...> after
...>   from_after = true
...> end
iex> from_try
** (RuntimeError) undefined function: from_try/0
iex> from_after
** (RuntimeError) undefined function: from_after/0
```

至此我们结束了对```try/catch/rescue```等知识的介绍。你会发现其实这些概念在实际的Elixir编程中不太常用。尽管的确有时也会用到。

是时候讨论一些Elixir的概念如理解（comprehensions）和魔法印（sigils）了。






















