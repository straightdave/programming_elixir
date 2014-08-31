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
注意```10 / 2```返回了一个浮点型的5.0而非整型的5。这是预期的。在Elixir中，```/```运算总是返回浮点型。
如果你想进行整型除法，或者求余数，可以使用方法```div```和```rem```。（rem的意思是division remainder）：
```
iex> div(10, 2)
5
iex> div 10, 2
5
iex> rem 10, 3
1
```
注意在写方法参数时，括号是可选的。（ruby程序员会心一笑）

Elixir支持用**捷径**书写二进制、八进制、十六进制整数，如：
```
iex> 0b1010
10
iex> 0o777
511
iex> 0x1F
31
```
揉揉眼，八进制是```0o```，数字0+小写o。

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


