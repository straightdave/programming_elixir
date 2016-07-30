4-模式匹配
==========

本章起教程进入 _不那么基础的_ 阶段，开始涉及函数式编程概念。
对之前没有函数式编程经验的人来说，这一章是一个基础，需要好好学习和理解。   

在Elixir中，```=```运算符实际上叫做 *匹配运算符*。
本章将讲解如何使用```=```运算符来对各种数据结构进行模式匹配。
最后本章还会讲解pin运算符(```^```)，用来访问某变量之前绑定的值。

## 4.1-匹配运算符

我们已经多次使用```=```符号进行变量的赋值操作：
```elixir
iex> x = 1
1
iex> x
1
```

在Elixir中，```=```作为 *匹配运算符*。下面来学习这样的概念：
```elixir
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

注意```1 = x```是一个合法的表达式。
由于前面的例子给x赋值为1，因此在匹配时左右相同，所以它匹配成功了。而两侧不匹配的时候，MatchError将被抛出。

变量只有在匹配操作符```=```的左侧时才被赋值：
```elixir
iex> 1 = unknown
** (RuntimeError) undefined function: unknown/0
```
错误原因是unknown变量没有被赋过值，Elixir猜你想调用一个名叫```unknown/0```的函数，
但是找不到这样的函数。

>
变量名在等号左边，Elixir认为是赋值表达式；变量名放在右边，Elixir认为是拿该变量的值和左边的值做匹配。

## 4.2-模式匹配
匹配运算符不光可以匹配简单数值，还能用来 *解构* 复杂的数据类型。例如，我们在元组上使用模式匹配：
```elixir
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

在两端不匹配的情况下，模式匹配会失败。比方说，匹配的两端的元组不一样长：
```elixir
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

或者两端模式有区别（比如两端数据类型不同）：
```elixir
iex> {a, b, c} = [:hello, "world", "!"]
** (MatchError) no match of right hand side value: [:hello, "world", "!"]
```

利用“匹配”的这个概念，我们可以匹配特定值；或者在匹配成功时，为某些变量赋值。   

下面例子中先写好了匹配的左端，它要求右端必须是个元组，且第一个元素是原子```:ok```。
```elixir
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13

iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

用在列表上：
```elixir
iex> [a, 2, 3] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

列表支持匹配自己的```head```和```tail```
（这相当于同时调用```hd/1```和```tl/1```，给```head```和```tail```赋值）：
```elixir
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

同```hd/1```和```tl/1```函数一样，以上代码不能对空列表使用：
```elixir
iex> [h|t] = []
** (MatchError) no match of right hand side value: []
```

>
[head|tail]这种形式不光在模式匹配时可以用，还可以用作向列表插入前置数值：
```elixir
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0|list]
[0, 1, 2, 3]
```

模式匹配使得程序员可以容易地解构数据结构（如元组和列表）。
在后面我们还会看到，它是Elixir的一个基础，对其它数据结构同样适用，比如图和二进制。

小结：
* 模式匹配使用```=```符号
* 匹配中等号左右的“模式”必须相同
* 变量在等号左侧才会被赋值
* 变量在右侧时必须有值，Elixir拿这个值和左侧相应位置的元素做匹配


## 4.3-pin运算符
在Elixir中，变量可以被重新绑定：
```elixir
iex> x = 1
1
iex> x = 2
2
```
>Elixir可以给变量重新绑定（赋值）。
它带来一个问题，就是对一个单独变量（而且是放在左端）做匹配时，
Elixir会认为这是一个重新绑定（赋值）操作，而不会当成匹配，执行匹配逻辑。
这里就要用到pin运算符。

如果你不想这样，可以使用pin运算符(^)。
加上了pin运算符的变量，在匹配时使用的值是本次匹配前就赋予的值：
```elixir
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {x, ^x} = {2, 1}
{2, 1}
iex> x
2
```

注意如果一个变量在匹配中被引用超过一次，所有的引用都应该绑定同一个模式：
```elixir
iex> {x, x} = {1, 1}
1
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

有些时候，你并不在意模式匹配中的一些值。
可以把它们绑定到特殊的变量 “ _ ” (underscore)上。
例如，如果你只想要某列表的head，而不要tail值。你可以这么做：
```elixir
iex> [h | _ ] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

变量“ _ ”特殊之处在于它不能被读，尝试读取它会报“未绑定的变量”错误：
```elixir
iex> _
** (CompileError) iex:1: unbound variable _
```

尽管模式匹配看起来如此牛逼，但是语言还是对它的作用做了一些限制。
比如，你不能让函数调用作为模式匹配的左端。下面例子就是非法的：
```elixir
iex> length([1,[2],3]) = 3
** (CompileError) iex:1: illegal pattern
```

模式匹配介绍完了。 在以后的章节中，模式匹配是常用的语法结构。
