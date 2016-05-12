21-Erlang库
============

Elixir对Erlang库提供了完善的交互。实际上，Elixir不是简单地去对Erlang库进行语言包装，
而是直接连接Erlang的代码。本章将展示一些Elixir中没有，但是常用（常见+有用）的Erlang功能。

随着对Elixir更加深入的学习和使用，你可能会更多地参考Erlang的
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

## The digraph module

[The digraph module](http://erlang.org/doc/man/digraph.html) (as well as
[digraph_utils](http://erlang.org/doc/man/digraph_utils.html)) contains
functions for dealing with directed graphs built of vertices and edges.
After constructing the graph, the algorithms in there will help finding
for instance the shortest path between two vertices, or loops in the graph.

Note that the functions in `:digraph` alter the graph structure indirectly
as a side effect, while returning the added vertices or edges.

Given three vertices, find the shortest path from the first to the last.

```iex
iex> digraph = :digraph.new()
iex> coords = [{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
iex> [v0, v1, v2] = (for c <- coords, do: :digraph.add_vertex(digraph, c))
iex> :digraph.add_edge(digraph, v0, v1)
iex> :digraph.add_edge(digraph, v1, v2)
iex> :digraph.get_short_path(digraph, v0, v2)
[{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
```

## Erlang Term Storage

The modules [`ets`](http://erlang.org/doc/man/ets.html) and
[`dets`](http://erlang.org/doc/man/dets.html) handle storage of large
data structures in memory or on disk respectively.

ETS lets you create a table containing tuples. By default, ETS tables
are protected, which means only the owner process may write to the table
but any other process can read. ETS has some functionality to be used as
a simple database, a key-value store or as a cache mechanism.

The functions in the `ets` module will modify the state of the table as a
side-effect.

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

## The math module

[The `math` module](http://erlang.org/doc/man/math.html) contains common
mathematical operations covering trigonometry, exponential and logarithmic
functions.

```iex
iex> angle_45_deg = :math.pi() * 45.0 / 180.0
iex> :math.sin(angle_45_deg)
0.7071067811865475
iex> :math.exp(55.0)
7.694785265142018e23
iex> :math.log(7.694785265142018e23)
55.0
```

## The queue module

The [`queue` is a data structure](http://erlang.org/doc/man/queue.html)
that implements (double-ended) FIFO (first-in first-out) queues efficiently:

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

## The rand module

[`rand` has functions](http://erlang.org/doc/man/rand.html) for returning
random values and setting the random seed.

```iex
iex> :rand.uniform()
0.8175669086010815
iex> _ = :rand.seed(:exs1024, {123, 123534, 345345})
iex> :rand.uniform()
0.5820506340260994
iex> :rand.uniform(6)
6
```

## The zip and zlib modules

[The `zip` module](http://erlang.org/doc/man/zip.html) lets you read and write zip files to and from disk or memory,
as well as extracting file information.

This code counts the number of files in a zip file:

```iex
iex> :zip.foldl(fn _, _, _, acc -> acc + 1 end, 0, :binary.bin_to_list("file.zip"))
{:ok, 633}
```

[The `zlib` module](http://erlang.org/doc/man/zlib.html) deals with data compression in zlib format, as found in the
`gzip` command.

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
