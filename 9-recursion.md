9-递归
======
因为在Elixir中（或所有函数式语言中），数据有不变性（immutability），因此在写循环时与传统的命令式（imperative）语言有所不同。
例如某命令式语言的循环可以这么写：
```
for(i = 0; i < array.length; i++) {
  array[i] = array[i] * 2
}
```

上面例子中，我们改变了```array```，以及辅助变量```i```的值。这在Elixir中是不可能的。
尽管如此，函数式语言却依赖于某种形式的循环---递归：一个函数可以不断地被递归调用，直到某条件满足才停止。
考虑下面的例子：打印一个字符串若干次：
```
defmodule Recursion do
  def print_multiple_times(msg, n) when n <= 1 do
    IO.puts msg
  end

  def print_multiple_times(msg, n) do
    IO.puts msg
    print_multiple_times(msg, n - 1)
  end
end

Recursion.print_multiple_times("Hello!", 3)
# Hello!
# Hello!
# Hello!
```

一个函数可以有许多子句（上面看起来定义了两个函数，但卫兵条件不同，可以看作同一个函数的两个子句）。
当参数匹配该子句的模式，且该子句的卫兵表达式返回true，才会执行该子句内容。
上面例子中，当```print_multiple_times/2```第一次调用时，n的值是3。

第一个子句有卫兵表达式要求n必须小于等于1。因为不满足此条件，代码找该函数的下一条子句。

参数匹配第二个子句，且该子句也没有卫兵表达式，因此得以执行。
首先打印```msg```，然后调用自身并传递第二个参数```n-1```(=2)。
这样```msg```又被打印一次，之后调用自身并传递参数```n-1```(=1)。

这个时候，n满足第一个函数子句条件。遂执行该子句，打印```msg```，然后就此结束。

我们称例子中第一个函数子句这种子句为“基本情形”。
基本情形总是最后被执行，因为起初通常都不匹配执行条件，程序而转去执行其它子句。
但是，每执行一次其它子句，条件都向这基本情形靠拢一点，直到最终回到基本情形处执行代码。

下面我们使用递归的威力来计算列表元素求和：
```
defmodule Math do
  def sum_list([head|tail], accumulator) do
    sum_list(tail, head + accumulator)
  end

  def sum_list([], accumulator) do
    accumulator
  end
end

Math.sum_list([1, 2, 3], 0) #=> 6
```

我们首先用列表[1,2,3]和初值0作为参数调用函数，程序将逐个匹配各子句的条件，执行第一个符合要求的子句。
于是，参数首先满足例子中定义的第一个子句。参数匹配使得head = 1，tail = [2,3]，accumulator = 0。

然后执行该字据内容，把head + accumulator作为第二个参数，连带去掉了head的列表做第一个参数，再次调用函数本身。
如此循环，每次都把新传入的列表的head加到accumulator上，传递去掉了head的列表。
最终传递的列表为空，符合第二个子句的条件，执行该子句，返回accumulator的值6。

几次函数调用情况如下：
```
sum_list [1, 2, 3], 0
sum_list [2, 3], 1
sum_list [3], 3
sum_list [], 6
```

这种使用列表做参数，每次削减一点列表的递归方式，称为“递减”算法，是函数式编程的核心。
<br/>
如果我们想给每个列表元素加倍呢？：
```
defmodule Math do
  def double_each([head|tail]) do
    [head * 2| double_each(tail)]
  end

  def double_each([]) do
    []
  end
end

Math.double_each([1, 2, 3]) #=> [2, 4, 6]
```

此处使用了递归来遍历列表元素，使它们加倍，然后返回新的列表。
这样以列表为参数，递归处理其每个元素的方式，称为“映射（map）”算法。
<br/>

递归和列尾调用优化（tail call optimization）是Elixir中重要的部分，通常用来创建循环。
尽管如此，在Elixir中你很少会使用以上方式来递归地处理列表。

下一章要介绍的[Enum模块](http://elixir-lang.org/docs/stable/elixir/Enum.html)为操作列表提供了诸多方便。
比如，下面例子：
```
iex> Enum.reduce([1, 2, 3], 0, fn(x, acc) -> x + acc end)
6
iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
[2, 4, 6]
```

或者，使用捕捉的语法：
```
iex> Enum.reduce([1, 2, 3], 0, &+/2)
6
iex> Enum.map([1, 2, 3], &(&1 * 2))
[2, 4, 6]
```


























