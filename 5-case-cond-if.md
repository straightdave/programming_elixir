5-流程控制
==========
[case](#51-case) <br/>
[卫兵子句中的表达式](#52-%E5%8D%AB%E5%85%B5%E5%AD%90%E5%8F%A5%E4%B8%AD%E7%9A%84%E8%A1%A8%E8%BE%BE%E5%BC%8F) <br/>
[cond](#53-cond) <br/>
[if和unless](#54-if%E5%92%8Cunless) <br/>
[do语句块](#55-do%E8%AF%AD%E5%8F%A5%E5%9D%97) <br/>

本章讲解case，cond和if的流程控制结构。

## 5.1-case
case将一个值与许多模式进行比较，直到找到一个匹配的：
```
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
```

如果与一个已赋值的变量做比较，要用pin运算符(^)标记该变量：
```
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Won't match"
...>   _  -> "Will match"
...> end
```

可以加上卫兵子句（guard clauses）提供额外的条件：
```
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Won't match"
...> end
```
于是上面例子中，第一个待比较的模式多了一个条件：x必须是正数。

## 5.2-卫兵子句中的表达式
Erlang中只允许以下表达式出现在卫兵子句中：
  - 比较运算符（==，!=，===，!==，>，<，<=，>=）
  - 布尔运算符（and，or）以及否定运算符（not，!）
  - 算数运算符（+，-，*，/）
  - <>和++如果左端是字面值
  - in运算符
  - 以下类型判断函数：
    - is_atom/1
    - is_binary/1
    - is_bitstring/1
    - is_boolean/1
    - is_float/1
    - is_function/1
    - is_function/2
    - is_integer/1
    - is_list/1
    - is_map/1
    - is_number/1
    - is_pid/1
    - is_reference/1
    - is_tuple/1
  - 外加以下函数：
    - abs(number)
    - bit_size(bitstring)
    - byte_size(bitstring)
    - div(integer, integer)
    - elem(tuple, n)
    - hd(list)
    - length(list)
    - map_size(map)
    - node()
    - node(pid | ref | port)
    - rem(integer, integer)
    - round(number)
    - self()
    - tl(list)
    - trunc(number)
    - tuple_size(tuple)

记住，卫兵子句中出现的错误不会漏出，只会简单地让卫兵条件失败：
```
iex> hd(1)
** (ArgumentError) argument error
    :erlang.hd(1)
iex> case 1 do
...>   x when hd(x) -> "Won't match"
...>   x -> "Got: #{x}"
...> end
"Got 1"
```

如果case中没有一条模式能匹配，会报错：
```
iex> case :ok do
...>   :error -> "Won't match"
...> end
** (CaseClauseError) no case clause matching: :ok
```

匿名函数也可以像下面这样，用多个模式或卫兵条件来灵活地匹配该函数的参数：
```
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```
需要注意的是，所有case模式中表示的参数个数必须一致，否则会报错。
上面的例子两个待匹配模式都是x，y。如果再有一个模式表示的参数是x，y，z，那就不行：
```
iex(5)> f2 = fn
...(5)>   x,y -> x+y
...(5)>   x,y,z -> x+y+z
...(5)> end
** (CompileError) iex:5: cannot mix clauses with different arities in function definition
    (elixir) src/elixir_translator.erl:17: :elixir_translator.translate/2
```

## 5.3-cond
case是拿一个值去同多个值或模式进行匹配，匹配了就执行那个分支的语句。
然而，许多情况下我们要检查不同的条件，找到第一个结果为true的，执行它的分支。
这时我们用cond：
```
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```
这样的写法和命令式语言里的```else if```差不多一个意思（尽管很少这么写）。

如果没有一个条件结果为true，会报错。因此，实际应用中通常会使用true作为最后一个条件。
因为即使上面的条件没有一个是true，那么该cond表达式至少还可以执行这最后一个分支：
```
iex> cond do
...>   2 + 2 == 5 ->
...>     "This is never true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   true ->
...>     "This is always true (equivalent to else)"
...> end
```
用法就好像许多语言中switch语句中的default一样。

最后需要注意的是，cond视所有除了false和nil的数值都为true：
```
iex> cond do
...>   hd([1,2,3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

## 5.4 if和unless
除了case和cond，Elixir还提供了两很常用的宏：if/2和unless/2，用它们检查单个条件：
```
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

如果给if/2的条件结果为false或者nil，那么它在do/end间的语句块就不会执行，该表达式返回nil。
unless/2相反。

它们都支持else语句块：
```
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```

>有趣的是，if/2和unless/2是以宏的形式提供的，而不像在很多语言中那样是语句。
可以阅读文档或if/2的源码（[Kernel模块](http://elixir-lang.org/docs/stable/elixir/Kernel.html)）。_Kernel_模块还定义了诸如+/2运算符和is_function/2函数。它们是默认是被自动导入，因而在你的代码中可用。

## 5.5-```do```语句块
以上讲解的4种流程控制结构：case，cond，if和unless，它们都被包裹在do/end语句块中。
即使我们把if语句写成这样：
```
iex> if true, do: 1 + 2
3
```

在Elixir中，do/end语句块方便地将一组表达式传递给```do:```。以下是等同的：
```
iex> if true do
...>   a = 1 + 2
...>   a + 10
...> end
13
iex> if true, do: (
...>   a = 1 + 2
...>   a + 10
...> )
13
```
我们称第二种语法使用了**关键字列表（keyword lists）**。我们可以这样传递else：
```
iex> if false, do: :this, else: :that
:that
```

注意一点，do/end语句块永远是被绑定在最外层的函数调用上。例如：
```
iex> is_number if true do
...>  1 + 2
...> end
```
将被解析为：
```
iex> is_number(if true) do
...>  1 + 2
...> end
```
这使得Elixir认为你是要调用函数is_number/2（第一个参数是if true，第二个是语句块）。
这时就需要加上括号解决二义性：
```
iex> is_number(if true do
...>  1 + 2
...> end)
true
```
关键字列表在Elixir语言中占有重要地位，在许多函数和宏中都有使用。后文中还会对其进行详解。














