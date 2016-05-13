21-Erlang库
============

Elixir对Erlang库提供了完善的交互。实际上，Elixir不是简单地去对Erlang库进行语言包装，
而是直接连接Erlang的代码（因为同源，Elixir以特定语法来直接调用Erlang库函数）。
本章将展示一些Elixir中没有，但是常用（常见+有用）的Erlang功能函数。

>随着对Elixir更加深入的学习和使用，你可能会更多地阅读参考Erlang的
[标准库手册](http://erlang.org/doc/apps/stdlib/index.html)。

## 二进制串模块

内建的Elixir字符串模块处理UTF-8编码过的二进制串（binaries）。而Erlang的
[二进制串模块](http://erlang.org/doc/man/binary.html)可能对你更加有用，
因为它可以处理的二进制串不一定非要是UTF-8编码的：

```iex
iex> String.to_char_list "Ø"
[216]
iex> :binary.bin_to_list "Ø"
[195, 152]
```

从上面的例子你就能看出一些区别来了；`String`模块返回UTF-8的字符码，
而`:binary`是原始的数据字节。

## 格式化的字符串输出

Elixir中并没有类似于C中的`printf`函数。作为一个可选项，你可以使用字符串插值来完成同样的功能：

```iex
iex> f = Float.to_string(:math.pi, decimals: 3) |> String.rjust(10)
iex> str = "Pi is approximately given by: #{f}"
"Pi is approximately given by:      3.142"
```

另外，还可以用Erlang标准库中的`:io.format/2`和`:io_lib.format/2`函数。
第一个格式化后输出到终端，而第二个输出到一个iolist。具体格式化的语法和`pringf`略有区别，
详见[Erlang文档](http://erlang.org/doc/man/io.html#format-1)：

```iex
iex> :io.format("Pi is approximately given by:~10.3f~n", [:math.pi])
Pi is approximately given by:     3.142
:ok
iex> to_string :io_lib.format("Pi is approximately given by:~10.3f~n", [:math.pi])
"Pi is approximately given by:     3.142\n"
```

另外需要注意的是Erlang的格式化函数中对Unicode的处理。

## 日历模块

[日历模块](http://erlang.org/doc/man/calendar.html) 包含本地时间和标准时间的转换函数，
以及其它一些时间函数。

```iex
iex> :calendar.day_of_the_week(1980, 6, 28)
6
iex> {date, time} = :calendar.now_to_local_time(:erlang.timestamp)
iex> date
{2016, 2, 17}
iex> time
{22, 4, 55}
```

## 加密模块

[加密模块](http://erlang.org/doc/man/crypto.html) 包含哈希方法，数字签名，
加密等功能函数：

```iex
iex> Base.encode16(:crypto.hash(:sha256, "Elixir"))
"3315715A7A3AD57428298676C5AE465DADA38D951BDFAC9348A8A31E9C7401CB"
```

`:crypto` 模块不是Erlang的标准库，但是包含在了Erlang发行包中。
这要求你必须在项目的配置文件中列出`:crypto`模块作为依赖项。
通过修改`mix.exs`来加载该模块：

```elixir
  def application do
    [applications: [:crypto]]
  end
```

## 有向图模块

[有向图模块](http://erlang.org/doc/man/digraph.html) （以及
[有向图模块工具](http://erlang.org/doc/man/digraph_utils.html)）
包含了处理有向图（有丁点和边构成）的函数。
创建了一个图实例之后，模块的算法即可帮助找寻顶点最短路径、图中的环等。

给出三个顶点，找寻从第一个到最后一个顶点的最短路径：

```iex
iex> digraph = :digraph.new()
iex> coords = [{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
iex> [v0, v1, v2] = (for c <- coords, do: :digraph.add_vertex(digraph, c))
iex> :digraph.add_edge(digraph, v0, v1)
iex> :digraph.add_edge(digraph, v1, v2)
iex> :digraph.get_short_path(digraph, v0, v2)
[{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
```

## ETS（Erlang Term Storage）：Erlang的“Term存储”机制

模块[`ets`](http://erlang.org/doc/man/ets.html)以及
[`dets`](http://erlang.org/doc/man/dets.html)
分别处理在内存中、磁盘上存储大型数据结构。

ETS创建一个表来存储元祖。默认情况下，ETS表是受保护的：只有owner进程才能写表，
其它进程只可以读。ETS提供了一些功能，可被当做简单的数据库、键值存储或cache机制使用。

作为副作用，`ets`模块中的函数会修改表的状态。

```iex
iex> table = :ets.new(:ets_test, [])
# Store as tuples with {name, population}
iex> :ets.insert(table, {"China", 1_374_000_000})
iex> :ets.insert(table, {"India", 1_284_000_000})
iex> :ets.insert(table, {"USA", 322_000_000})
iex> :ets.i(table)
<1   > {"USA", 322000000}
<2   > {"China", 1_374_000_000}
<3   > {"India", 1_284_000_000}
```

## 数学模块

[数学模块](http://erlang.org/doc/man/math.html) 包含了常用数学操作，
如三角函数、指数或底数函数等等。

```iex
iex> angle_45_deg = :math.pi() * 45.0 / 180.0
iex> :math.sin(angle_45_deg)
0.7071067811865475
iex> :math.exp(55.0)
7.694785265142018e23
iex> :math.log(7.694785265142018e23)
55.0
```

## 队列（queue）模块

[队列 `queue` 是一种数据结构](http://erlang.org/doc/man/queue.html)
实现了双向先进先出的高效率队列：

```iex
iex> q = :queue.new
iex> q = :queue.in("A", q)
iex> q = :queue.in("B", q)
iex> {value, q} = :queue.out(q)
iex> value
{:value, "A"}
iex> {value, q} = :queue.out(q)
iex> value
{:value, "B"}
iex> {value, q} = :queue.out(q)
iex> value
:empty
```

## 随机值（rand）模块

[`rand`模块函数](http://erlang.org/doc/man/rand.html) 可以返回随机值或是设置随机seed：

```iex
iex> :rand.uniform()
0.8175669086010815
iex> _ = :rand.seed(:exs1024, {123, 123534, 345345})
iex> :rand.uniform()
0.5820506340260994
iex> :rand.uniform(6)
6
```

## zip和zlib模块

[`zip`模块](http://erlang.org/doc/man/zip.html) 可以让你读写磁盘或内存中的zip文件，
以及提取其文件信息等。

以下代码计算了zip压缩包中的文件数量：

```iex
iex> :zip.foldl(fn _, _, _, acc -> acc + 1 end, 0, :binary.bin_to_list("file.zip"))
{:ok, 633}
```

[`zlib`模块](http://erlang.org/doc/man/zlib.html) 处理zlib格式的数据压缩
（如gzip中用的）：

```iex
iex> song = "
...> Mary had a little lamb,
...> His fleece was white as snow,
...> And everywhere that Mary went,
...> The lamb was sure to go."
iex> compressed = :zlib.compress(song)
iex> byte_size song
110
iex> byte_size compressed
99
iex> :zlib.uncompress(compressed)
"\nMary had a little lamb,\nHis fleece was white as snow,\nAnd everywhere that Mary went,\nThe lamb was sure to go."
```
