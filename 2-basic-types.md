2-基本类型
==========
[基本算数运算](#21-%E5%9F%BA%E6%9C%AC%E7%AE%97%E6%95%B0%E8%BF%90%E7%AE%97) <br/>
[布尔]() <br/>
[原子]() <br/>
[字符串]() <br/>
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

