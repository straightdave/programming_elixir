2-基本类型
==========
[基本算数运算](#21-%E5%9F%BA%E6%9C%AC%E7%AE%97%E6%95%B0%E8%BF%90%E7%AE%97) <br/>
[布尔](#22-%E5%B8%83%E5%B0%94) <br/>
[原子](#23-%E5%8E%9F%E5%AD%90) <br/>
[字符串](#24-%E5%AD%97%E7%AC%A6%E4%B8%B2) <br/>
[匿名函数]() <br/>
[（链式）列表]() <br/>
[元组]() <br/>
[列表还是元组？]() <br/>
<br/>
本章介绍Elixir一些基本类型，如：整型，浮点型，布尔，原子，字符串，列表等等。
一些类型如：
```
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
打开```iex```输入以下命令
```
iex> 1 + 2
3
iex> 5 * 5
25
iex> 10 / 2
5.0
```
>```10 / 2```返回了一个浮点型的5.0而非整型的5。这是预期的。在Elixir中，```/```运算符总是返回浮点型。

如果你想进行整型除法，或者求余数，可以使用函数```div```和```rem```。（rem的意思是division remainder）：
```
iex> div(10, 2)
5
iex> div 10, 2
5
iex> rem 10, 3
1
```
>在写函数参数时，括号是可选的。（ruby程序员会心一笑）

Elixir支持用**捷径**书写二进制、八进制、十六进制整数，如：
```
iex> 0b1010
10
iex> 0o777
511
iex> 0x1F
31
```
>揉揉眼，八进制是```0o```，数字0+小写o。

输入浮点型数字需要一个小数点，且在其后至少有一位数字。Elixir支持使用```e```来表示指数。
```
iex> 1.0
1.0
iex> 1.0e-10
1.0e-10
```
Elixir中浮点型都是64位精度。

## 2.2-布尔
Elixir使用```true```和```false```两个布尔值。
```
iex> true
true
iex> true == false
false
```
Elixir提供了许多用以判断类型的函数，如```is_boolean/1```函数可以用来检查参数是不是布尔型。
>在Elixir中，函数通过名称和参数个数（又称元数arity）来识别。如```is_boolean/1```表示名为```is_boolean```，接受一个参数的函数；而```is_boolean/2```表示同名，但接受2个参数的不同函数（只是大哥比方，这样的is_boolean其实不存在）。
>另外，```is_boolean/1```这样的表述，实际上是为了在讲述函数时方便，在实际程序中，是不用写```/1```或是```/2```的。

```
iex> is_boolean(true)
true
iex> is_boolean(1)
false
```
同样的函数还有```is_integer/1```，```is_float/1```，```is_number/1```，分别测试参数是否是整型、浮点型或者两者其一。
>可以在交互式命令行中使用```h```命令来打印函数或运算符的帮助信息。如```h is_boolean/1```或```h ==/2```。注意提及函数时不但要给出名称还要加上元数```/<arity>```。

## 2.3-原子
原子是一种常量，它的名字就是它的值。有些语言中称其为**符号（symbol）**（ruby程序员会心一笑）。
```
iex> :hello
:hello
iex> :hello == :world
false
```
布尔值```true```和```false```实际上就是原子：
```
iex> true == :true
true
iex> is_atom(false)
true
```

## 2.4-字符串
在Elixir中，字符串以**双括号**包裹。它们都是UTF-8编码。
```
iex> "hellö"
"hellö"
```
Elixir支持字符串插值：
```
iex> "hellö #{:world}"
"hellö world"
```
（ruby程序员会心一笑）
<br/>
字符串可以直接包含换行符，或者它的转义字符：
```
iex> "hello
...> world"
"hello\nworld"
iex> "hello\nworld"
"hello\nworld"
```
你可以使用```IO```模块（module）里的```IO.puts/1```方法打印字符串：
```
iex> IO.puts "hello\nworld"
hello
world
:ok
```
>IO.puts/1函数返回原子值```:ok```。起名叫puts，ruby程序员表示好熟悉。

字符串在Elixir内部被表示为二进制数值，也就是一串bytes：
```
iex> is_binary("hellö")
true
```

你可以查看字符串包含的byte数量：
```
iex> byte_size("hellö")
6
```
>为啥是6？不是5个字符么？注意里面有一个非ASCII字符```ö```，在UTF-8下被编码为2个bytes。

我们可以使用专门的函数来返回字符串中的字符数：
```
iex> String.length("hellö")
5
```

在[String模块](http://elixir-lang.org/docs/stable/elixir/String.html)中提供了很多定义在Unicode标准上的函数来操作字符串。，如：
```
iex> String.upcase("hellö")
"HELLÖ"
```
记住，单引号和双引号包裹的字符串在Elixir中是两种不同的数据类型。
```
iex> 'hellö' == "hellö"
false
```
我们将在之后关于“二进制、字符串与字符列表”章节中详细讲述。

## 2.5-匿名函数
在Elixir中，使用关键字```fn```和```end```来界定函数。如：
```
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
在Elixir中，函数是**头等公民**。你可以将函数作为参数传递给其他函数，就像整型和浮点型一样。在上面的例子中，我们向函数```is_function/1```传递了由变量```add```表示的匿名函数，返回```true```。我们还可以调用函数```is_function/2```来判断参数函数的元数（参数个数）。

注意，在调用一个匿名函数时，在变量名和写参数的括号之间要有个**点号(.)**。

匿名函数是闭包，意味着它们可以访问它们被定义的范围（scope）内的其它变量：
```
iex> add_two = fn a -> add.(a, 2) end
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> add_two.(2)
4
```
但是注意，在匿名函数内修改了所引用的外部变量的值，并不实际反映到该变量上：
```
iex> x = 42
42
iex> (fn -> x = 0 end).()
0
iex> x
42
```

## 2.6-（链式）列表
Elixir使用方括号标识列表。列表可以包含任意类型的值：
```
iex> [1, 2, true, 3]
[1, 2, true, 3]
iex> length [1, 2, 3]
3
```

两个列表可以使用```++/2```拼接，使用```--/2```做“减法”：
```
iex> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex> [1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

本教程的讲解将多次涉及列表的头（下文使用head表述）和尾（下文使用tail表述）的概念。
列表的head指的是第一个元素，而tail指的是除了第一个元素剩下的元素。它们分别可以用函数```hd/1```和```tl/1```取出：
```
iex> list = [1,2,3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
```
尝试从一个空列表中取出head或tail将会报错：
```
iex> hd []
** (ArgumentError) argument error
```

## 2.7-元组
Elixir使用大括号（花括号）定义元组（tuples）。类似列表，元组也可以承载任意类型的数据。：
```
iex> {:ok, "hello"}
{:ok, "hello"}
iex> tuple_size {:ok, "hello"}
2
```
元组使用连续的内存空间存储数据。这意味着可以很方便地使用索引访问元组数据，或者获取元组大小。（索引从0开始）：
```
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
iex> tuple_size(tuple)
2
```

也可以很方便地使用函数```put_elem/3```设置某个位置的元素值：
```
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> put_elem(tuple, 1, "world")
{:ok, "world"}
iex> tuple
{:ok, "hello"}
```

注意函数```put_elem/3```返回一个新元组。原来那个由tuple标识的元组没有被改变。这是因为Elixir的数据类型是**不可变的**。这种不可变性使你永远不用担心你的数据结构在某处被某些代码改变。
在处理并发程序时，该不可变性还有利于减少多个不同程序实体在同时修改一个数据结构时引起的竞争以及其他麻烦。

## 2.8-列表还是元组？
列表与元组的区别。列表在内存中是以链表的形式存储的，一个元素指向下一个元素，然后再下一个...直到到达列表末尾。我们称这样的一对（元素值+指向下一个元素的指针）为列表的一个单元（cons cell）。用Elixir语法表示这种模式：
```
iex> list = [1|[2|[3|[]]]]
[1, 2, 3]
```
>列表方括号中的竖线（|）表示列表head与tail的分界。

这种模式意味着获取列表的长度是一个线性操作：我们必须遍历完整个列表才能知道它的长度。而且前置拼接操作很快捷：
```
iex> [0] ++ list
[0, 1, 2, 3]
iex> list ++ [4]
[1, 2, 3, 4]
```
上面的第一条语句是前置拼接操作，执行起来很快。因为它简单地添加了一个指向原先整个列表的新列表单元，原先的列表没有变化。
而第二条语句较慢，因为它需要重建整个链表，让原先列表的最后一个元素指向那个新元素。

另一方面，元组在内存中是连续存储的。这意味着获取元组的大小或者使用索引访问元组元素十分快速。
但是，修改或添加元素操作开销很大，因为它们涉及在内存中对元组的整体复制。

这些关于性能特性的讨论揭示了如何去使用这两个不同的数据结构。函数常用元组来返回多个信息。如```File.read/1```，它读取文件内容，返回元组：
```
iex> File.read("path/to/existing/file")
{:ok, "... contents ..."}
iex> File.read("path/to/unknown/file")
{:error, :enoent}
```

如果传递给```File.read/1```的文件路径有效，那么函数返回元组，它的首元素是原子```:ok```，第二个元素是文件内容。
否则将返回第一个元素是```:error```，第二个元素是错误信息的元组。

大多数情况下，Elixir会引导你做正确的事。如有个叫```elem/2```的函数，它使用索引来访问一个元组元素。
但是它没有相应的列表版本，因为列表的机制不适用通过索引来访问：
```
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
```

当需要计算某数据结构包含的元素个数时，Elixir遵循一个简单的规则：如果操作在常数时间内完成（如数值是提前算好的），这样的函数通常被命名为_size_。而如果操作需要显式地计数，那么该函数通常命名为_length_。

例如，目前讲到过的4个计数函数：```byte_size/1```（用来计算字符串有多少bytes），```tuple_size/1```（用来计算元组大小），```length/1```（计算列表长度）以及```String.length/1```（计算字符串中的字符数）。
按照命名规则，当我们用```byte_size```获取字符串大小时，开销较小。但是当我们用```String.length```获取字符串unicode字符个数时，需要遍历整个字符串。

Elixir还提供了**Port**，**Reference**和**PID**三个数据类型（常用于进程交互），这些将在讲解进程时讲到。


Elixir








