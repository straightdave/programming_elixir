6-二进制串、字符串和字符列表
========================

在“基本类型”一章中，介绍了字符串，以及使用`is_binary/1`函数检查它：

```elixir
iex> string = "hello"
"hello"
iex> is_binary string
true
```

本章将学习理解：二进制串（binaries）是个啥，它怎么和字符串（strings）扯上关系的；
以及用单引号包裹的值`'like this'`是啥意思。

## UTF-8和Unicode

字符串是UTF-8编码的二进制串。
为了弄清这句话的准确含义，我们要先理解两个概念：字节（bytes）和字符编码（code point）的区别。
Unicode标准为我们已知的大部分字母分配了字符编码。
比如，字母`a`的字符编码是`97`，而字母`ł`的字符编码是`322`。
当把字符串`"hełło"`写到硬盘上的时候，需要将字符编码转化为字节。
如果我们遵循一个字节表示一个字符编码这个，那是写不了`"hełło"`的。
因为字母`ł`的编码是`322`，而一个字节所能存储的数值范围是`0`到`255`。
但是如你所见，确实能够在屏幕上显示`"hełło"`，说明还是有*某种*解决方法的，于是*编码*便出现了。

要用字节表示字符编码，我们需要用某种方式对其进行编码。
Elixir选择UTF-8为主要并且默认的编码方式。
当我们说某个字符串是UTF-8编码的二进制串，指的是该字符串是一串字节，
这些字节以某种方式（即UTF-8编码）组织起来，表示特定的字符编码。

因为给字母`ł`分配的字符编码是`322`，因此在实际上需要一个以上的字节来表示。
这就是为什么我们会看到，调用函数`byte_size/1`和`String.length/1`的结果不一样：

```elixir
iex> string = "hełło"
"hełło"
iex> byte_size string
7
iex> String.length string
5
```

>注意：如果你使用Windows，你的终端有可能不是默认使用UTF-8编码方式。你需要在进入`iex(iex.bat)`之前，
首先执行`chcp 65001`命令来修改当前Session的编码方式。

UTF-8需要1个字节来表示`h`，`e`，`o`的字符编码，用2个字节表示`ł`。
在Elixir中可以使用`?`运算符获取字符的编码：

```elixir
iex> ?a
97
iex> ?ł
322
```

你还可以使用
[String](http://elixir-lang.org/docs/stable/elixir/String.html)模块里的函数
将字符串切成单独的字符编码：

```elixir
iex> String.codepoints("hełło")
["h", "e", "ł", "ł", "o"]
```

Elixir为字符串操作提供了强大的支持，它支持Unicode的许多操作。实际上，Elixir通过了文章
[“字符串类型崩坏了”](http://mortoray.com/2013/11/27/the-string-type-is-broken/)
记录的所有测试。

然而，字符串只是故事的一小部分。如果字符串正如所言是二进制串，那我们使用`is_binaries/1`函数时，
Elixir必须一个底层类型来支持字符串。事实亦如此，下面就来介绍这个底层类型---二进制串。

## 二进制串（以及比特串`bitstring`）

在Elixir中可以用`<<>>`定义一个二进制串：

```elixir
iex> <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> byte_size(<<0, 1, 2, 3>>)
4
```

一个二进制串只是一连串的字节而已。这些字节可以以任何方式组织，即使凑不成一个合法的字符串：

```elixir
iex> String.valid?(<<239, 191, 191>>)
false
```

而字符串的拼接操作实际上就是二进制串的拼接操作：

```elixir
iex> <<0, 1>> <> <<2, 3>>
<<0, 1, 2, 3>>
```

一个常见技巧是，通过给一个字符串尾部拼接一个空（null）字节`<<0>>`，
可以看到该字符串内部二进制串的样子：

```elixir
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

二进制串中的每个数值都表示一个字节，其数值最大范围是255。
二进制允许使用修改器显式标注一下那个数值的存储空间大小，使其可以存储超过255的数值；
或者将一个字符编码转换为utf8编码后的形式（变成多个字节的二进制串）：

```elixir
iex> <<255>>
<<255>>
iex> <<256>> # 被截断（truncated）
<<0>>
iex> <<256 :: size(16)>> # 使用16比特（bits）即2个字节来保存
<<1, 0>>
iex> <<256 :: utf8>> # 这个数字是一个字符的编码，将其使用utf8方式编码为字节
"Ā"    # 注意，在iex交互窗口中，所有可以作为合法字符串的二进制串，都会显示为字符串
iex> <<256 :: utf8, 0>> # 尾部拼接个空字节，查看上一条命令结果内部实际的二进制串
<<196, 128, 0>>
```

如果一个字节是8个比特，那如果我们给一个大小是1比特的修改器会怎样？：

```elixir
iex> <<1 :: size(1)>>
<<1::size(1)>>
iex> <<2 :: size(1)>> # 被截断（truncated）
<<0::size(1)>>
iex> is_binary(<< 1 :: size(1)>>) # 二进制串失格
false
iex> is_bitstring(<< 1 :: size(1)>>)
true
iex> bit_size(<< 1 :: size(1)>>)
1
```

这样（每个元素长度是1比特）就不再是二进制串（人家每个元素是一个字节，起码8比特），
退化成为比特串（bitstring），意思就是一串比特！
所以，所以，二进制串就是一特殊的比特串，比特总数是8的倍数。

也可以对二进制串或比特串做模式匹配：

```elixir
iex> <<0, 1, x>> = <<0, 1, 2>>
<<0, 1, 2>>
iex> x
2
iex> <<0, 1, x>> = <<0, 1, 2, 3>>
** (MatchError) no match of right hand side value: <<0, 1, 2, 3>>
```

注意，在没有修改器标识的情况下，二进制串中的每个元素都应该匹配8个比特长度。
因此上面最后的例子，匹配的左右两端不具有相同容量，因此出现错误。

下面是使用了修改器标识的匹配例子：

```elixir
iex> <<0, 1, x :: binary>> = <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> x
<<2, 3>>
```

上面例子使用了`binary`修改器，指示`x`是个二进制串。（为啥不用单词的复数形式`binaries`搞不懂啊。）
这个修改器仅仅可以用在被匹配的串的*末尾元素*上。

跟上面例子同样的原理，使用字符串的连接操作符`<>`，效果相似：

```elixir
iex> "he" <> rest = "hello"
"hello"
iex> rest
"llo"
```

A complete reference about the binary / bitstring constructor <<>> can be found in the Elixir documentation. This concludes our tour of bitstrings, binaries and strings. A string is a UTF-8 encoded binary and a binary is a bitstring where the number of bits is divisible by 8. Although this shows the flexibility Elixir provides for working with bits and bytes, 99% of the time you will be working with binaries and using the is_binary/1 and byte_size/1 functions.

关于二进制串/比特串的构造器`<< >>`完整的参考，
请见[Elixir的文档](http://elixir-lang.org/docs/stable/elixir/Kernel.SpecialForms.html#%3C%3C%3E%3E/1)。

总之，记住字符串是UTF-8编码后的二进制串，而二进制串是特殊的、元素数量是8的倍数的比特串。
尽管这种机制增加了Elixir在处理比特或字节时的灵活性，
而现实中99%的时候你只会用到`is_binary/1`和`byte_size/1`函数跟二进制串打交道。

## 字符列表（char lists）

字符列表就是字符的列表。

```elixir
iex> 'hełło'
[104, 101, 322, 322, 111]
iex> is_list 'hełło'
true
iex> 'hello'
'hello'
```

可以看出，比起包含字节，一个字符列表包含的是单引号所引用的一串字符各自的字符编码。
注意IEx遇到超出ASCII值范围的字符编码时，显示其字符编码的值，而不是字符。
双引号引用的是字符串（即二进制串），单引号表示的是字符列表（即，一个列表）。

实际应用中，字符列表常被用来做为同一些Erlang库交互的参数，因为这些老库不接受二进制串作为参数。
要将字符列表和字符串之间相互转换，可以使用函数`to_string/1`和`to_char_list/1`：

```elixir
iex> to_char_list "hełło"
[104, 101, 322, 322, 111]
iex> to_string 'hełło'
"hełło"
iex> to_string :hello
"hello"
iex> to_string 1
"1"
```

注意这些函数是多态的。它们不但可以将字符列表转化为字符串，还能转化整数、原子等为字符串。
