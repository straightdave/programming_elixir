11-进程
=======

Elixir里所有代码都在进程中执行。进程彼此独立，并发执行，通过传递消息（message）进行沟通。
进程不仅仅是Elixir并发编程的基础，也是Elixir创建分布式、高容错程序的本质。

Elixir的进程和传统操作系统中的进程不可混为一谈。
Elixir的进程在CPU和内存使用上，是极度轻量级的（但不同于其它语言中的线程）。
正因如此，同时运行着数十万、百万个进程也并不是罕见的事。

本章将讲解如何派生新进程，以及在不同进程间发送和接受消息等基本知识。

## 进程派生（spawning）

派生（spawning）一个新进程的方法是使用自动导入（kernel模块）的```spawn/1```函数：

```elixir
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

函数```spawn/1```接收一个_函数_作为参数，在其派生出的进程中执行这个函数。

注意，```spawn/1```返回一个PID（进程标识）。在这个时候，这个派生的进程很可能已经结束。

派生的进程执行完函数后便会结束：

```elixir
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

>你可能会得到与例子中不一样的PID

用```self/0```函数获取当前进程的PID：
```elixir
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

>注：上文调用```self/0```加了括号。
但是如前文所说，在不引起误解的情况下，可以省略括号而只写```self```

可以发送和接收消息，让进程变得越来越有趣。

## 发送和接收（消息）

使用```send/2```函数发送消息，用```receive/1```接收消息：

```elixir
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

当有消息被发给某进程，该消息就被存储在该进程的“邮箱”里。
语句块```receive/1```检查当前进程的邮箱，寻找匹配给定模式的消息。
函数```receive/1```支持卫兵语句（guards）及分支子句（clause）如```case/2```。

如果找不到匹配的消息，当前进程将一直等待，直到下一条信息到达。

可以给等待设置一个超时时间：
```elixir
iex> receive do
...>   {:hello, msg} -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

超时时间设为0表示你知道当前邮箱内肯定有邮件存在，很自信，因此设了这个极短的超时时间。

把以上概念综合起来，演示进程间发送消息：
```elixir
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

在shell中执行程序时，辅助函数```flush/0```很有用。它清空并打印进程邮箱中的所有消息：
```elixir
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## 链接（links）

Elixir中最常用的进程派生方式是通过调用函数```spawn_link/1```。

在举例子讲解```spawn_link/1```之前，来看看如果一个进程挂掉会发生什么：
```elixir
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Process #PID<0.58.00> raised an exception
** (RuntimeError) oops
    :erlang.apply/2
```

...仅仅是打印了一些文字信息，而派生这个倒霉进程的父进程仍然全无知觉地继续运行。
这是因为进程是互不干扰的。
如果我们希望一个进程发生异常挂掉可以被另一个进程感知，我们需要链接它们。

这需要使用```spawn_link/1```函数来派生进程，例子：
```elixir
iex> spawn_link fn -> raise "oops" end
#PID<0.60.0>
** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

当这个失败发生在shell中，shell会自动捕获这个异常并显示格式优雅的异常信息。
为了弄明白失败时真正会发生什么，让我们在一个脚本文件中使用```spawn_link/1```：

```elixir
# spawn.exs
spawn_link fn -> raise "oops" end

receive do
  :hello -> "let's wait until the process fails"
end
```

执行：
```elixir
$ elixir spawn.exs

** (EXIT from #PID<0.47.0>) an exception was raised:
    ** (RuntimeError) oops
        spawn.exs:1: anonymous fn/0 in :elixir_compiler_0.__FILE__/1
```

这次，该进程在失败时把它的父进程也弄停止了，因为它们是链接的。

使用函数```spawn_link/1```是在派生时对父子进程进行链接，
你还可以手动对两个进程进行链接：使用函数```Process.link/1```。
我们推荐可以多看看[Process模块](http://elixir-lang.org/docs/stable/elixir/Process.html)，里面包含很多常用的进程操作函数。

进程和链接在创建能高容错系统时扮演重要角色。
在Elixir程序中，我们经常把进程链接到“管理者（supervisors）”上。
由这个角色负责检测失败进程，并且创建新进程取代之。这是唯一可行的方式。
因为进程间独立，默认情况下不共享任何东西。一个进程失败了，不会影响其它进程的状态。

其它语言通常需要我们抛出/捕获异常，而在Elixir中我们可以放任进程挂掉，
因为我们希望“管理者”会以更合适的方式重启系统。
“死快一点（failing fast）”是Elixir软件开发中的一个常见哲学。

`spawn/1`和`spawn_link/1`是Elixir中创建进程的基本方式。
尽管我们到目前为止都是专门调用它们，实际上大部分时间我们会使用基于它们功能的一些抽象操作。
比如常见的：“任务（Tasks）”。

## 任务（Task）

任务建立在进程派生函数之上，提供了更好的错误报告和内省。

```elixir
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
** (RuntimeError) oops
    (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
    (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []
```

我们使用```Task.start/1```和```Task.start_link/1```代替```spawn/1```和```spawn_link/1```，
返回```{:ok pid}```而不仅仅是子进程的PID。这使得任务可以在监督者树中使用。
另外，任务提供了许多便利的函数，比如```Task.async/1```和```Task.await/1```等，
以及其它一些易于构建分布式接结构的函数。

我们将在《高级》篇中介绍关于任务更多的函数功能。现在我们只需要知道，使用任务有更好的错误报告。

## 状态（state）

目前为止我们还没有怎么谈到状态。但是如果你需要构建一个程序它需要状态，比如保存程序的配置信息，
或者解析一个文件先把它保存在内存里，你在哪儿存储？

_进程_就是（最常见的一个）答案。我们可以写个进程执行无限循环，保存若干状态，
通过收发消息维护和改变这些状态值。
举个例子，我们写一个程序模块，创建一个提供键值对存储服务的进程：

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end

```

注意```start_link```函数启动一个新的进程。这个进程以一个空的图（map）为参数，执行```loop/1```函数。
启动后，```loop/1```函数等待消息，并且针对每个消息执行合适的操作。
如果收到```:get```消息，它会把消息发回给调用者，然后再次调用自身```loop/1```，等待新消息。
如果收到```:put```消息，它便用一个新版本的图变量（里面保存了新的键/值）再次调用```loop/1```。

执行一下```iex kv.exs```：
```elixir
iex> {:ok, pid} = KV.start_link
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
nil
```

一开始进程内的图变量是没有键值的，所以发送一个```:get```消息并且刷新当前进程的收件箱，返回nil。
下面发送一个```:put```消息再试一次：

```elixir
iex> send pid, {:put, :hello, :world}
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

注意这个进程是怎么保持状态信息的：我们通过向该进程发送消息来获取和更新状态。
事实上，其它任何进程只要知道这个进程的PID，都能读取和修改状态。

还可以注册这个PID，给它一个名称。这使得我们可以通过名字来向它发送消息：
```elixir
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

使用进程维护状态、进程名称注册都是构建Elixir应用的常见模式。
但是大多数时间我们不会自己实现这样的模式，而是使用Elixir已经提供的抽象实现。

例如，Elixir提供的[agent](http://elixir-lang.org/docs/stable/elixir/Agent.html)就是一个状态维护进程的简单实现：

```elixir
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

给```Agent.start_link/2```方法加上```:name```选项，可以自动为其注册一个名字。

除了agents，Elixir还提供了一套API来创建通用服务器（generic servers，称作GenServer），任务等。
这些都是建立在进程概念之上的实现。其它概念，包括“管理者”树，都可以在《高级》篇里找到更详细的说明。

下一章将介绍Elixir语言的I/O世界。
