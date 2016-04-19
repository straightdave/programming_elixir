2-基本类型
==========  

本章介绍Elixir的基本类型。Elixir主要的基本类型有：
整型（integer），浮点（float），布尔（boolean），原子（atom，又称symbol符号），
字符串（string），列表（list）和元组（tuple）等。   

它们在iex中显示如下：
```elixir
iex> 1          # integer
iex> 0x1F       # integer
iex> 1.0        # float
iex> true       # boolean
iex> :atom      # atom / symbol
iex> "elixir"   # string
iex> [1, 2, 3]  # list
iex> {1, 2, 3}  # tuple
```

## 2.1-基本算数运算
打开```iex```，输入以下表达式：
```elixir
iex> 1 + 2
3
iex> 5 * 5
25
iex> 10 / 2
5.0
```
>```10 / 2```返回了一个浮点型的5.0而非整型的5，这是预期的。   
在Elixir中，```/```运算符总是返回浮点型数值。

如果你想进行整型除法，或者求余数，可以使用函数```div```和```rem```。
（rem的意思是division remainder，余数）：
```elixir
iex> div(10, 2)
5
iex> div 10, 2
5
iex> rem 10, 3
1
```
>在写函数参数时，括号是可选的。（ruby程序员会心一笑）

Elixir支持用 **捷径（shortcut）** 书写二进制、八进制、十六进制整数。如：
```elixir
iex> 0b1010
10
iex> 0o777
511
iex> 0x1F
31
```
>揉揉眼，八进制是```0o```，数字0 + 小写o。

输入浮点型数字需要一个小数点，且在其后至少有一位数字。
Elixir支持使用```e```来表示指数：
```elixir
iex> 1.0
1.0
iex> 1.0e-10
1.0e-10
```
Elixir中浮点型都是64位双精度。

## 2.2-布尔
Elixir使用```true```和```false```两个布尔值。
```elixir
iex> true
true
iex> true == false
false
```
Elixir提供了许多用以判断类型的函数，如```is_boolean/1```函数可以用来检查参数是不是布尔型。

>在Elixir中，函数通过名称和参数个数（又称元数，arity）来识别。
如```is_boolean/1```表示名为```is_boolean```，接受一个参数的函数；
而```is_boolean/2```表示与其同名、但接受2个参数的**不同**函数。（只是打个比方，这样的is_boolean实际上不存在）   
另外，```<函数名>/<元数>```这样的表述是为了在讲述函数时方便，在实际程序中如果调用函数，
是不用注明```/1```或```/2```的。

```elixir
iex> is_boolean(true)
true
iex> is_boolean(1)
false
```

类似的函数还有```is_integer/1```，```is_float/1```，```is_number/1```，
分别测试参数是否是整型、浮点型或者两者其一。

>可以在交互式命令行中使用```h```命令来打印函数或运算符的帮助信息。
如```h is_boolean/1```或```h ==/2```。
注意此处提及某个函数时，不但要给出名称，还要加上元数```/<arity>```。

## 2.3-原子
原子（atom）是一种常量，名字就是它的值。
有些语言中称其为 **符号（symbol）**（如ruby）：
```elixir
iex> :hello
:hello
iex> :hello == :world
false
```

布尔值```true```和```false```实际上就是原子：
```elixir
iex> true == :true
true
iex> is_atom(false)
true
```

## 2.4-字符串
在Elixir中，字符串以 **双括号** 包裹，采用UTF-8编码：
```elixir
iex> "hellö"
"hellö"
```

Elixir支持字符串插值（和ruby一样使用```#{ ... }```）：
```elixir
iex> "hellö #{:world}"
"hellö world"
```

字符串可以直接包含换行符，或者其转义字符：
```elixir
iex> "hello
...> world"
"hello\nworld"
iex> "hello\nworld"
"hello\nworld"
```

你可以使用```IO```模块（module）里的```IO.puts/1```方法打印字符串：
```elixir
iex> IO.puts "hello\nworld"
hello
world
:ok
```
函数```IO.puts/1```打印完字符串后，返回原子值```:ok```。

字符串在Elixir内部被表示为二进制数值（binaries），也就是一连串的字节（bytes）：
```elixir
iex> is_binary("hellö")
true
```
>注意，二进制数值（binary）是Elixir内部的存储结构之一。
字符串、列表等类型在语言内部就表示为二进制数值，因此它们也可以被专门操作二进制数值的函数修改。

你可以查看字符串包含的字节数量：
```elixir
iex> byte_size("hellö")
6
```
>为啥是6？不是5个字符么？注意里面有一个非ASCII字符```ö```，在UTF-8下被编码为2个字节。

我们可以使用专门的函数来返回字符串中的字符数量：
```elixir
iex> String.length("hellö")
5
```

[String模块](http://elixir-lang.org/docs/stable/elixir/String.html)中提供了
很多符合Unicode标准的函数来操作字符串。如：
```elixir
iex> String.upcase("hellö")
"HELLÖ"
```

记住，单引号和双引号包裹的字符串在Elixir中是两种不同的数据类型：
```elixir
iex> 'hellö' == "hellö"
false
```
我们将在之后关于“二进制、字符串与字符列表”章节中详细讲述它们的区别。

## 2.5-匿名函数
在Elixir中，使用关键字```fn```和```end```来界定函数。如：
```elixir
iex> add = fn a, b -> a + b end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> is_function(add)
true
iex> is_function(add, 2)
true
iex> is_function(add, 1)
false
iex> add.(1, 2)
3
```
在Elixir中，函数是 **一等公民**。你可以将函数作为参数传递给其他函数，就像整型和浮点型一样。
在上面的例子中，我们向函数```is_function/1```传递了由变量```add```表示的匿名函数，
结果返回```true```。
我们还可以调用函数```is_function/2```来判断该参数函数的元数（参数个数）。

注意，在调用一个匿名函数时，在变量名和写参数的括号之间要有个 **点号(.)**。

匿名函数是闭包，意味着它们可以保留其定义的作用域（scope）内的其它变量值：
```elixir
iex> add_two = fn a -> add.(a, 2) end
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> add_two.(2)
4
```
这个例子定义的匿名函数```add_two```它内部使用了之前在同一个iex内定义好的```add```变量。
但要注意，在匿名函数内修改了所引用的外部变量的值，并不实际反映到该变量上：
```elixir
iex> x = 42
42
iex> (fn -> x = 0 end).()
0
iex> x
42
```
这个例子中匿名函数把引用了外部变量x，并修改它的值为0。这时函数执行后，外部的x没有被影响。

## 2.6-（链式）列表
Elixir使用方括号标识列表。列表可以包含任意类型的值：
```elixir
iex> [1, 2, true, 3]
[1, 2, true, 3]
iex> length [1, 2, 3]
3
```

两个列表可以使用```++/2```拼接，使用```--/2```做“减法”：
```elixir
iex> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex> [1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

本教程将多次涉及列表的头（head）和尾（tail）的概念。
列表的头指的是第一个元素，而尾指的是除了第一个元素以外，其它元素组成的列表。
它们分别可以用函数```hd/1```和```tl/1```从原列表中取出：
```elixir
iex> list = [1,2,3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
```

尝试从一个空列表中取出头或尾将会报错：
```elixir
iex> hd []
** (ArgumentError) argument error
```

## 2.7-元组
Elixir使用大括号（花括号）定义元组（tuples）。类似列表，元组也可以承载任意类型的数据。：
```elixir
iex> {:ok, "hello"}
{:ok, "hello"}
iex> tuple_size {:ok, "hello"}
2
```
元组使用连续的内存空间存储数据。
这意味着可以很方便地使用索引访问元组数据，以及获取元组大小。（索引从0开始）：
```elixir
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
iex> tuple_size(tuple)
2
```

也可以很方便地使用函数```put_elem/3```设置某个位置的元素值：
```elixir
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> put_elem(tuple, 1, "world")
{:ok, "world"}
iex> tuple
{:ok, "hello"}
```

注意函数```put_elem/3```返回一个新元组。原来那个由变量tuple标识的元组没有被改变。
这是因为Elixir的数据类型是 **不可变的**。
这种不可变性使你永远不用担心你的数据会在某处被某些代码改变。
在处理并发程序时，这种不可变性有利于减少多个程序实体同时修改一个数据结构时引起的竞争以及其他麻烦。

## 2.8-列表还是元组？
列表与元组的区别：列表在内存中是以链表的形式存储的，一个元素指向下一个元素，
然后再下一个...直到到达列表末尾。
我们称这样的一对（元素值+指向下一个元素的指针）为列表的一个单元（cons cell）。   

用Elixir语法表示这种模式：
```elixir
iex> list = [1|[2|[3|[]]]]
[1, 2, 3]
```
>列表方括号中的竖线（|）表示列表头与尾的分界。

这个原理意味着获取列表的长度是一个线性操作：我们必须遍历完整个列表才能知道它的长度。
但是列表的前置拼接操作很快捷：
```elixir
iex> [0] ++ list
[0, 1, 2, 3]
iex> list ++ [4]
[1, 2, 3, 4]
```

上面例子中第一条语句是前置拼接操作，执行起来很快。
因为它只是简单地添加了一个新列表单元，它指向原先列表头部。而原先的列表没有任何变化。
第二条语句是后缀拼接操作，执行速度较慢。
这是因为它 **重建** 了原先的列表，让原先列表的末尾元素指向那个新元素。

而另一方面，元组在内存中是连续存储的。
这意味着获取元组大小，或者使用索引访问元组元素的操作十分快速。
但是元组在修改或添加元素时开销很大，因为这些操作会在内存中对元组的进行整体复制。

这些讨论告诉我们当如何在不同的情况下选择使用不同的数据结构。

函数常用元组来返回多个信息。如```File.read/1```，它读取文件内容，返回一个元组：
```elixir
iex> File.read("path/to/existing/file")
{:ok, "... contents ..."}
iex> File.read("path/to/unknown/file")
{:error, :enoent}
```

如果传递给函数```File.read/1```的文件路径有效，那么函数返回一个元组，
其首元素是原子```:ok```，第二个元素是文件内容。
如果路径无效，函数也将返回一个元组，其首元素是原子```:error```，第二个元素是错误信息。

大多数情况下，Elixir会引导你做正确的事。
如有个叫```elem/2```的函数，它使用索引来访问一个元组元素。
这个函数没有相应的列表版本，因为根据存储机制，列表不适用通过索引来访问：
```elixir
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
```

当需要计算某数据结构包含的元素个数时，Elixir遵循一个简单的规则：
如果操作在常数时间内完成（答案是提前算好的），这样的函数通常被命名为 _*size_。
而如果操作需要显式计数，那么该函数通常命名为 _*length_。

例如，目前讲到过的4个计数函数：```byte_size/1```
（用来计算字符串有多少字节），```tuple_size/1```
（用来计算元组大小），```length/1```（计算列表长度）
以及```String.length/1```（计算字符串中的字符数）。

按照命名规则，当我们用```byte_size```获取字符串所占字节数时，开销较小。
但是当我们用```String.length```获取字符串unicode字符个数时，需要遍历整个字符串，开销较大。

除了本章介绍的数据类型，Elixir还提供了 **Port**，**Reference** 和 **PID** 三个数据类型（它们常用于进程交互）。这些数据类型将在讲解进程时详细介绍。
