4-模式匹配
==========
[匹配运算符]() <br/>
[模式匹配]() <br/>
[pin运算符]() <br/>

本章起教程进入不那么基础的阶段，开始涉及函数式编程概念。在Elixir中，=运算符实际上是一个匹配运算符。
本章将讲解如何使用=运算符来对数据结构进行模式匹配。最后本章还会讲解pin运算符(^)，用来访问某变量之前绑定的值。

## 4.1-匹配运算符
我们已经多次使用=符号进行变量的赋值操作：
```
iex> x = 1
1
iex> x
1
```
在Elixir中，=号其实称为匹配运算符。下面来学习这样的概念：
```
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

注意```1 = x```是一个合法的表达式。由于前面给x赋值为1，左右相同，所以它匹配成功了。而两侧不匹配的时候，MatchError将被抛出。

变量只有在匹配操作符=的左侧时才被赋值：
```
iex> 1 = unknown
** (RuntimeError) undefined function: unknown/0
```
错误原因是unknown变量没有被赋过值，Elixir猜你想调用一个名叫unknown/0的函数，但是找不到这样的函数。

## 4.2-模式匹配
匹配运算符不光可以匹配简单数值，还能用来解构复杂的数据类型。例如，我们在元组上使用模式匹配：
```
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```
在两端不匹配的情况下，模式匹配会失败。比方说，匹配两端的元组不一样长：
```
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

或者两端不是一个类型：
```
iex> {a, b, c} = [:hello, "world", "!"]
** (MatchError) no match of right hand side value: [:hello, "world", "!"]
```

有趣的是，我们可以匹配特定值。下面例子中匹配的左端当且仅当右端是个元组，且第一个元素是原子:ok。
```
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13

iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

用在列表上：
```
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

列表支持匹配自己的head和tail（这相当于同时调用hd/1和tl/1，给head和tail赋值）：
```
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

同hd/1和tl/1函数一样，以上代码不能对空列表使用：
```
iex> [h|t] = []
** (MatchError) no match of right hand side value: []
```

>[head|tail]这种形式不光在模式匹配时可以用，还可以用作前置数值：
```
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0|list]
[0, 1, 2, 3]
```

模式匹配使得程序员可以容易地解构数据结构（如元组和列表）。在后面我们还会看到，它是Elixir的一个基础，对其它数据结构同样适用，比如图和二进制。

## 4.3-pin运算符
在Elixir中，变量可以被重新绑定：
```
iex> x = 1
1
iex> x = 2
2
```

如果你不想这样，可以使用pin运算符(^)。加上了pin运算符的变量，在匹配时使用的值是匹配前就赋予的值：
```
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
```
iex> {x, x} = {1, 1}
1
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

有些时候，你并不在意模式里的一些值。通常你就可以把它们绑定到特殊的变量“_”上。例如，如果你只想要某列表的head，而不要tail值。你可以这么做：
```
iex> [h | _] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

变量“_”特殊之处在于它不能被读。尝试读取它会报“为绑定的变量”错误：
```
iex> _
** (CompileError) iex:1: unbound variable _
```

尽管模式匹配可以让我们创建功能强大的结构，但是它的作用被限制了。
比如，你不能让函数调用作为匹配的左端。下面例子就是非法的：
```
iex> length([1,[2],3]) = 3
** (CompileError) iex:1: illegal pattern
```








