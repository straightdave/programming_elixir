21-Erlang库
============

Elixir provides excellent interoperability with Erlang libraries. In fact,
Elixir discourages simply wrapping Erlang libraries in favor of directly
interfacing with Erlang code. In this section we will present some of the
most common and useful Erlang functionality that is not found in Elixir.

As you grow more proficient in Elixir, you may want to explore the Erlang
[STDLIB Reference Manual](http://erlang.org/doc/apps/stdlib/index.html) in more
detail.

## The binary module

The built-in Elixir String module handles binaries that are UTF-8 encoded.
[The binary module](http://erlang.org/doc/man/binary.html) is useful when
you are dealing with binary data that is not necessarily UTF-8 encoded.

```iex
iex> String.to_char_list "Ø"
[216]
iex> :binary.bin_to_list "Ø"
[195, 152]
```

The above example shows the difference; the `String` module returns UTF-8
codepoints, while `:binary` deals with raw data bytes.

## Formatted text output

Elixir does not contain a function similar to `printf` found in C and other
languages. An option is to rely on string interpolation to achieve a similar
result:

```iex
iex> f = Float.to_string(:math.pi, decimals: 3) |> String.rjust(10)
iex> str = "Pi is approximately given by: #{f}"
"Pi is approximately given by:      3.142"
```

Alternatively, the Erlang standard library functions `:io.format/2` and
`:io_lib.format/2` may be used. The first formats to terminal output, while
the second formats to an iolist. The format specifiers differ from `printf`,
[refer to the Erlang documentation for details](http://erlang.org/doc/man/io.html#format-1).

```iex
iex> :io.format("Pi is approximately given by:~10.3f~n", [:math.pi])
Pi is approximately given by:     3.142
:ok
iex> to_string :io_lib.format("Pi is approximately given by:~10.3f~n", [:math.pi])
"Pi is approximately given by:     3.142\n"
```

Also note that Erlang's formatting functions require special attention to
Unicode handling.

## The calendar module

[The calendar module](http://erlang.org/doc/man/calendar.html) contains
functions for conversion between local and universal time, as well as
time conversion functions.

```iex
iex> :calendar.day_of_the_week(1980, 6, 28)
6
iex> {date, time} = :calendar.now_to_local_time(:erlang.timestamp)
iex> date
{2016, 2, 17}
iex> time
{22, 4, 55}
```

## The crypto module

[The crypto module](http://erlang.org/doc/man/crypto.html) contains hashing
functions, digital signatures, encryption and more:

```iex
iex> Base.encode16(:crypto.hash(:sha256, "Elixir"))
"3315715A7A3AD57428298676C5AE465DADA38D951BDFAC9348A8A31E9C7401CB"
```

The `:crypto` module is not part of the Erlang standard library, but is
included with the Erlang distribution. This means you must list `:crypto`
in your project's applications list whenever you use it. To do this,
edit your `mix.exs` file to include:

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
