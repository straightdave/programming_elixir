6-二进制-字符串-字符列表
========================
[UTF-8和Unicode]() <br/>
[二进制（和bitstring）]()<br/>
[字符列表]() <br/>

在“基本类型”一章中，介绍了字符串，以及使用is_binary/1函数检查它：
```
iex> string = "hello"
"hello"
iex> is_binary string
true
```

本章将学习理解，二进制（binaries）是个啥，它怎么和字符串扯上关系的，以及用单引号包裹的值，```'like this'```，是啥意思。

## 6.1-UTF-8和Unicode
字符串是UTF-8编码的二进制。为了弄清这句话啥意思，我们要先理解两个概念：bytes和code point的区别。
字母```a```的code point是97，而字母```ł```的code point是322。
当把字符串```"hełło"```写到硬盘上的时候，需要将其code point转化为bytes。
如果一个byte对应一个code point，那是写不了```"hełło"```的，因为字母```ł```的code point是322，超过了一个byte所能存储的最大数值（255）。
但是如你所见，该字母能够显示到屏幕上，说明还是有一定的解决方法的。于是_编码_便出现了。

要用byte表示code point，我们需要在一定程度上对其进行编码。
Elixir使用UTF-8为默认编码格式。
当我们说某个字符串是UTF-8编码的二进制数据，意思是该字符串是一串byte，以一定方法组织来表示特定的code points，即UTF-8编码。

因此当我们存储字母```ł```的时候，实际上是用两个bytes来表示它。
这就是为什么有时候对同一字符串调用函数```byte_size/1```和```String.length/1```结果不一样：
```
iex> string = "hełło"
"hełło"
iex> byte_size string
7
iex> String.length string
5
```

UTF-8需要1个byte来表示code points：h，e和o，用2个bytes表示ł。
在Elixir中可以使用```?```运算符获取code point值：
```
iex> ?a
97
iex> ?ł
322
```

你还可以使用[String模块](http://elixir-lang.org/docs/stable/elixir/String.html)里的函数
将字符串切成单独的code points：
```
iex> String.codepoints("hełło")
["h", "e", "ł", "ł", "o"]
```

Elixir为字符串操作提供了强大的支持。实际上，Elixir通过了文章[“字符串类型破了”](http://mortoray.com/2013/11/27/the-string-type-is-broken/)记录的所有测试。

不仅如此，因为字符串是二进制，Elixir还提供了更强大的底层类型的操作。下面就来介绍该底层类型---二进制。

## 6.2-二进制（和bitstring）
在Elixir中可以用```<<>>````定义一个二进制：
```
iex> <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> byte_size <<0, 1, 2, 3>>
4
```

一个二进制只是一连串bytes。这些bytes可以以任何方法组织，即使凑不成一个合法的字符串：
```
iex> String.valid?(<<239, 191, 191>>)
false
```

字符串的拼接操作实际上是二进制的拼接操作：
```
iex> <<0, 1>> <> <<2, 3>>
<<0, 1, 2, 3>>
```

一个常见技巧是，通过给某字符串尾部拼接一个null byte```<<0>>```，来看看该字符串内部二进制的样子：
```
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

二进制中的每个数值都表示一个byte，因此其最大是255。
如果超出了255，二进制允许你再提供一个修改器（标识一下那个位置的存储空间大小）使其可以存储；
或者将其转换为utf8编码后的形式（变成多个byte的二进制）：
```
iex> <<255>>
<<255>>
iex> <<256>> # truncated
<<0>>
iex> <<256 :: size(16)>> # use 16 bits (2 bytes) to store the number
<<1, 0>>
iex> <<256 :: utf8>> # the number is a code point
"Ā"
iex> <<256 :: utf8, 0>>
<<196, 128, 0>>
```

如果一个byte是8 bits，那如果我们给一个size是1 bit的修改器会怎样？：
```
iex> <<1 :: size(1)>>
<<1::size(1)>>
iex> <<2 :: size(1)>> # truncated
<<0::size(1)>>
iex> is_binary(<< 1 :: size(1)>>)
false
iex> is_bitstring(<< 1 :: size(1)>>)
true
iex> bit_size(<< 1 :: size(1)>>)
1
```
这样（每个元素是1 bit）就不再是二进制（人家每个元素是byte，至少8
 bits）了，而是bitstring，就是一串比特！
所以实际上二进制就是一串比特，只是比特数是8的倍数。









