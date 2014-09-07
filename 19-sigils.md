19-魔法印
==========
[正则表达式]() <br/>
[字符串、字符列表和单词魔法印]() <br/>
[自定义魔法印]() <br/>

我们已经学习了Elixir提供的双引号字符串和单引号的字符列表。但是对于Elixir中所有文本描述型数据结构来说，这只是冰山一角。
例如，原子就是一种。

Elixir的一个目的就是可扩展性：开发者能够为特定的领域来扩展语言。
计算机科学的领域已是如此广阔。几乎无法设计一门语言来涵盖其所有。
我们的打算是，与其创造出一种万能的语言，不如创造一种可扩展的语言，让开发者可以为其从事的领域对语言进行扩展。

本章将讲述“魔法印（sigils）”，它是Elixir提供的处理文本描述型数据的一种机制。

## 19.1-正则表达式
魔法印以波浪号（~）起头，后面跟着一个字母，然后是分隔符。
最常用的魔法印是~r，代表[正则表达式](https://en.wikipedia.org/wiki/Regular_Expressions)：
```
# A regular expression that returns true if the text has foo or bar
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

Elixir提供了Perl兼容的正则表达式（regex），由[PCRE库](http://www.pcre.org/)实现。

正则表达式支持修饰符（modifiers）。例如```i```修饰符使该正则表达式无视大小写：
```
iex> "HELLO" =~ ~r/hello/
false
iex> "HELLO" =~ ~r/hello/i
true
```

阅读[Regex模块](http://elixir-lang.org/docs/stable/elixir/Regex.html)获取关于其它修饰符的及其所支持的操作的更多信息。

目前为止，所有的例子都用了```/```界定正则表达式。事实上魔法印支持8种不同的分隔符：
```
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

支持多种分隔符是因为在处理不同的魔法印的时候更加方便。
比如，使用括号作为正则表达式的分隔符会让人困惑，分不清括号是正则模式的一部分还是别的什么。
但是，括号对其它魔法印来说很方便。

## 19.2-字符串、字符列表和单词魔法印
除了正则表达式，Elixir还提供了三种魔法印。

```~s``` 魔法印备用来生成字符串，类似双引号的作用：
```
iex> ~s(this is a string with "quotes")
"this is a string with \"quotes\""
```

而```~c```魔法印用来生成字符列表：
```
iex> ~c(this is a string with "quotes")
'this is a string with "quotes"'
```

```~w``` 魔法印用来生成单词，以空格分隔开：
```
iex> ~w(foo bar bat)
["foo", "bar", "bat"]
```

```~w``` 魔法印还接受```c```，```s```和```a```修饰符（分别代表字符列表，字符串和原子）来选择结果的类型：
```
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

除了小写的魔法印，Elixir还支持大写的魔法印。如，```~s```和```~S```都返回字符串，前者会解释转义字符而后者不会：
```
iex> ~s(String with escape codes \x26 interpolation)
"String with escape codes & interpolation"
iex> ~S(String without escape codes and without #{interpolation})
"String without escape codes and without \#{interpolation}"
```

字符串和字符列表支持一下转义字符：
  - \" 表示一个双引号
  - \' 表示一个单引号
  - \\ 表示一个反斜杠
  - \a 响铃
  - \b 退格
  - \d 删除
  - \e 退出
  - \f 换页
  - \n 新行
  - \r 换行
  - \s 空格
  - \t 水平制表符
  - \v 垂直制表符
  - \DDD, \DD, \D 八进制数字（如\377）
  - \xDD 十六进制数字（如\x13）
  - \x{D...} 多个十六进制字符的十六进制数（如\x{abc13}


魔法印还支持多行文本（heredocs），使用的是三个双引号或单引号：
```
iex> ~s"""
...> this is
...> a heredoc string
...> """
```

最常见的有多行文本的魔法印就是写注释文档了。
例如，如果你要在注释里写一些转义字符，这有可能会报错。
```
@doc """
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\\\"foo\\\"")
    "'foo'"

"""
def convert(...)
```

使用```~S```，我们就可以避免问题：
```
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

### 19.3-自定义魔法印
本章开头提到过，魔法印是可扩展的。事实上，魔法印```~r/foo/i```等于是用两个参数调用了函数```sigil_r```：
```
iex> sigil_r(<<"foo">>, 'i')
~r"foo"i
```
就是说，我们可以通过该函数读取魔法印```~r```的文档：
```
iex> h sigil_r
...
```

我们也可以通过实现相应的函数来提供我们自己的魔法印。例如，我们来实现```i(13)```魔法印，返回整数：
```
iex> defmodule MySigils do
...>   def sigil_i(string, []), do: String.to_integer(string)
...> end
iex> import MySigils
iex> ~i(13)
13
```
魔法印通过宏，可以用来做一些发生在*编译时*的工作。例如，正则表达式在编译时会被编译，而在执行的时候就不必再被编译了。
如果你对此主体感兴趣，可以多阅读关于宏的资料，并且阅读Kernel模块中那些魔法印的实现。
