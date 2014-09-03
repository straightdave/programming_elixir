10-枚举类型和流
==================
[枚举对象]() <br/>
[积极vs懒惰]() <br/>
[流]() <br/>

## 10.1-枚举类型
Elixir提供了枚举类型（enumerables）的概念，使用[Enum模块]()操作它们。我们已经介绍过两种枚举类型：列表和图。
```
iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]
```

Enum模块为枚举类型提供了大量函数来变化，排序，分组，过滤和读取元素。
Enum模块是开发者最常用的模块之一。
<br/>

Elixir还提供了范围（range）：
```
iex> Enum.map(1..3, fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.reduce(1..3, 0, &+/2)
6
```

因为Enum模块在设计时为了适用于不同的数据类型，所以它的API被限制为多数据类型适用的函数。
为了实现某些操作，你可能需要针对某类型使用某特定的模块。
比如，如果你要在列表中某特定位置插入一个元素，要用[List模块](http://elixir-lang.org/docs/stable/elixir/List.html)中的List.insert_at/3函数。而向某些类型内插入数据是没意义的，比如范围。

Enum中的函数是多态的，因为它们能处理不同的数据类型。
尤其是，模块中可以适用于不同数据类型的函数，它们是遵循了[Enumerable协议](http://elixir-lang.org/docs/stable/elixir/Enumerable.html)。
我们在后面章节中将讨论这个协议。下面将介绍一种特殊的枚举类型：流。

## 10.2-积极vs懒惰
Enum模块中的所有函数都是**积极**的。多数函数接受一个枚举类型，并返回一个列表：
```
iex> odd? = &(rem(&1, 2) != 0)
#Function<6.80484245/1 in :erl_eval.expr/5>
iex> Enum.filter(1..3, odd?)
[1, 3]
```

这意味着当使用Enum进行多种操作时，每次操作都生成一个中间列表，直到得出最终结果：
```
iex> 1..100_000 |> Enum.map(&(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
7500000000
```

上面例子是一个含有多个操作的管道。从一个范围开始，然后给每个元素乘以3。
该操作将会生成的中间结果是含有100000个元素的列表。
然后我们过滤掉所有偶数，产生又一个新中间结果：一个50000元素的列表。
最后求和，返回结果。

作为一个替代，[流模块](http://elixir-lang.org/docs/stable/elixir/Stream.html)提供了懒惰的实现：
```
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
7500000000
```

与之前Enum的处理不同，流先创建了一系列的计算操作。然后仅当我们把它传递给Enum模块，它才会被调用。流这种方式适用于处理大量的（甚至是无限的）数据集合。

## 10.3-流
流是懒惰的，比起Enum来说。
分步分析一下上面的例子，你会发现流与Enum的区别：
```
iex> 1..100_000 |> Stream.map(&(&1 * 3))
#Stream<[enum: 1..100000, funs: [#Function<34.16982430/1 in Stream.map/2>]]>
```
流操作返回的不是结果列表，而是一个数据类型---流，一个表示要对范围1..100000使用map操作的动作。

另外，当我们用管道连接多个流操作时：
```
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
#Stream<[enum: 1..100000, funs: [...]]>
```

流模块中的函数接受任何枚举类型为参数，返回一个流。
流模块还提供了创建流（甚至是无限操作的流）的函数。
例如，```Stream.cycle/1```可以用来创建一个流，它能无限周期性枚举所提供的参数（小心使用）：
```
iex> stream = Stream.cycle([1, 2, 3])
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
```

另一方面，```Stream.unfold/2```函数可以生成给定的有限值：
```
iex> stream = Stream.unfold("hełło", &String.next_codepoint/1)
#Function<39.75994740/2 in Stream.unfold/2>
iex> Enum.take(stream, 3)
["h", "e", "ł"]
```

另一个有趣的函数是```Stream.resource/3```，它可以用来包裹某资源，确保该资源在使用前打开，在用完后关闭（即使中途出现错误）。--类似C#中的use{}关键字。
比如，我们可以stream一个文件：
```
iex> stream = File.stream!("path/to/file")
#Function<18.16982430/2 in Stream.resource/3>
iex> Enum.take(stream, 10)
```

这个例子读取了文件的前10行内容。流在处理大文件，或者慢速资源（如网络）时非常有用。
<br/>

一开始Enum和流模块中函数的数量多到让人气馁。但你会慢慢地熟悉它们。
建议先熟悉Enum模块，然后因为应用而转去流模块中那些相应的，懒惰版的函数。
