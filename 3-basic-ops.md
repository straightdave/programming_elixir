3-基本运算符
============
通过前几章我们知道Elixir提供了 +，-，*，/ 4个算术运算符，外加整数除法函数div/2和取余函数rem/2。
Elixir还提供了++和--运算符来操作列表：
```
iex> [1,2,3] ++ [4,5,6]
[1,2,3,4,5,6]
iex> [1,2,3] -- [2]
[1,3]
```
使用<>进行字符串拼接：
```
iex> "foo" <> "bar"
"foobar"
```
Elixir还提供了三个布尔运算符：or，and和not。这三个运算符只接受布尔值作为第一个参数：
```
iex> true and true
true
iex> false or is_atom(:example)
true
```
如果提供了非布尔值作为第一个参数，会报异常：
```
iex> 1 and true
** (ArgumentError) argument error
```

or和and可短路，即它们仅在第一个参数无法决定整体结果的情况下才执行第二个参数：
```
iex> false and error("This error will never be raised")
false

iex> true or error("This error will never be raised")
true
```
>如果你是Erlang程序员，Elixir中的and和or其实就是andalso和orelse运算符。

除了这几个布尔运算符，Elixir还提供||，&&和!运算符。它们可以接受任意类型的参数值。
在使用这些运算符时，除了false和nil的值都被视作true：
```
# or
iex> 1 || true
1
iex> false || 11
11

# and
iex> nil && 13
nil
iex> true && 17
17

# !
iex> !true
false
iex> !1
false
iex> !nil
true
```

根据经验，当参数返回的是布尔时，使用and，or和not；如果非布尔值，用&&，||和!。

Elixir还提供了==，!=，===，!==，<=，>=，<和>这些比较运算符：
```
iex> 1 == 1
true
iex> 1 != 2
true
iex> 1 < 2
true
```

==和===的不同之处是后者在判断数字时更严格：
```
iex> 1 == 1.0
true
iex> 1 === 1.0
false
```

在Elixir中，可以判断不同类型数据的大小：
```
iex> 1 < :atom
true
```

这很实用。排序算法不必担心如何处理不同类型的数据。总体上，不同类型的排序顺序是：
```
number < atom < reference < functions < port < pid < tuple < maps < list < bitstring
```
不用背，只要知道有这么回事儿就可以。

