11-进程
=======
[进程派生](#111-%E8%BF%9B%E7%A8%8B%E6%B4%BE%E7%94%9F) <br/>
[发送和接收](#112-%E5%8F%91%E9%80%81%E5%92%8C%E6%8E%A5%E6%94%B6) <br/>
[链接](#113-%E9%93%BE%E6%8E%A5) <br/>
[状态](#114-%E7%8A%B6%E6%80%81) <br/>

Elixir中，所有代码都在进程中执行。进程彼此独立，一个接一个并发执行，彼此通过消息传递来沟通。
进程不仅仅是Elixir中最基本的并发单位，它们还是Elixir创建分布式和高容错程序的基础。

Elixir的进程和操作系统中的进程不可混为一谈。
Elixir的进程非常非常地轻量级（在使用CPU和内存角度上说。但是它又不同于其它语言中的线程）。
因此，同时运行着数万个进程也并不是罕见的事。

本章将讲解派生新进程的基本知识，以进程间如何发送和接受消息。

## 11.1-进程派生
派生（spawning）一个新进程的方法是使用自动导入（kernel函数）的```spawn/1```函数：
```elixir
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

函数```spawn/1```接收一个_函数_作为参数，在其派生出的进程中执行这个函数。

注意spawn/1返回一个PID（进程标识）。在这个时候，这个派生的进程很可能已经结束。
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

而可以发送和接收消息，让进程变得越来越有趣。

## 11.2-发送和接收
使用```send/2```函数发送消息，用```receive/1```接收消息：
```elixir
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg}  -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

当有消息被发给某进程，该消息就被存储在该进程的邮箱里。```receive/1```语句块检查当前进程的邮箱，寻找匹配给定模式的消息。
函数```receive/1```支持分支子句，如```case/2```。
当然也可以给子句加上卫兵表达式。

如果找不到匹配的消息，当前进程将一直等待，知道下一条信息到达。但是可以设置一个超时时间：
```elixir
iex> receive do
...>   {:hello, msg}  -> msg
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

在shell中执行程序时，辅助函数```flush/0```很有用。它清空缓冲区，打印进程邮箱中的所有消息：
```elixir
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## 11.3-链接
Elixir中最常用的进程派生方式是通过函数```spawn_link/1```。
在举例子讲解```spawn_link/1```之前，来看看如果一个进程失败了会发生什么：
```
iex> spawn fn -> raise "oops" end
#PID<0.58.0>
```

。。。啥也没发生。这时因为进程都是互不干扰的。如果我们希望一个进程中发生失败可以被另一个进程知道，我们需要链接它们。
使用```spawn_link/1```函数，例子：
```
iex> spawn_link fn -> raise "oops" end
#PID<0.60.0>
** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

当失败发生在shell中，shell会自动终止执行，并显示失败信息。这导致我们没法看清背后过程。
要弄明白链接的进程在失败时发生了什么，我们在一个脚本文件使用```spawn_link/1```并且执行和观察它：
```
# spawn.exs
spawn_link fn -> raise "oops" end

receive do
  :hello -> "let's wait until the process fails"
end
```

这次，该进程在失败时把它的父进程也弄停止了，因为它们是链接的。<br/>
手动链接进程：```Process.link/1```。
建议可以多看看[Process模块](http://elixir-lang.org/docs/stable/elixir/Process.html)，里面包含很多常用的进程操作函数。

进程和链接在创建能高容错系统时扮演重要角色。在Elixir程序中，我们经常把进程链接到某“管理者”上。
由这个角色负责检测失败进程，并且创建新进程取代之。因为进程间独立，默认情况下不共享任何东西。
而且当一个进程失败了，也不会影响其它进程。
因此这种形式（进程链接到“管理者”角色）是唯一的实现方法。
<br/>

其它语言通常需要我们来try-catch异常，而在Elixir中我们对此无所谓，放手任进程挂掉。
因为我们希望“管理者”会以更合适的方式重启系统。
“要死你就快一点”是Elixir软件开发的通用哲学。
<br/>

在讲下一章之前，让我们来看一个Elixir中常见的创建进程的情形。

## 11.4-状态
目前为止我们还没有怎么谈到状态。但是，只要你创建程序，就需要状态。
例如，保存程序的配置信息，或者分析一个文件先把它保存在内存里。
你怎么存储状态？


进程就是（最常见的）答案。我们可以写无限循环的进程，保存一个状态，然后通过收发信息来告知或改变该状态。
例如，写一个模块文件，用来创建一个提供k-v仓储服务的进程：
```
defmodule KV do
  def start do
    {:ok, spawn_link(fn -> loop(%{}) end)}
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

注意```start```函数简单地派生一个新进程，这个进程以一个空的图为参数，执行```loop/1```函数。
这个```loop/1```函数等待消息，并且针对每个消息执行合适的操作。
加入受到一个```:get```消息，它把消息发回给调用者，然后再次调用自身```loop/1```，等待新消息。
当受到```:put```消息，它便用一个新版本的图变量（里面的k-v更新了）再次调用自身。

执行一下试试：
```
iex> {:ok, pid} = KV.start
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
nil
```

一开始进程内的图变量是没有键值的，所以发送一个```:get```消息并且刷新当前进程的收件箱，返回nil。
下面再试试发送一个```:put```消息：
```
iex> send pid, {:put, :hello, :world}
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

注意进程是怎么保持一个状态的：我们通过同该进程收发消息来获取和更新这个状态。
事实上，任何进程只要知道该进程的PID，都能读取和修改状态。

还可以注册这个PID，给它一个名称。这使得人人都知道它的名字，并通过名字来向它发送消息：
```
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

使用进程维护状态，以及注册进程都是Elixir程序非常常用的方式。
但是大多数时间我们不会自己实现，而是使用Elixir提供的抽象实现。
例如，Elixir提供的[agent](http://elixir-lang.org/docs/stable/elixir/Agent.html)就是一种维护状态的简单的抽象实现：
```
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

给```Agent.start/2```方法加一个一个```:name```选项可以自动为其注册一个名字。
除了agents，Elixir还提供了创建通用服务器（generic servers，称作GenServer）、
通用时间管理器以及事件处理器（又称GenEvent）的API。
这些，连同“管理者”树，都可以在Mix和OTP手册里找到详细说明。

