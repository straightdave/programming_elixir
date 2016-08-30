18-速构（Comprehension）
========

> *Comprehensions* 翻译成“速构”是参照了《Erlang/OTP in Action》译者的用辞。
国内一些Python书（是的，Python也有这个概念）中翻译为“推导式”、“递推式”。
这里不用纠结它的翻译，更重要的是需要弄明白它是什么。

>“速构”是函数式语言中常见的概念，指的是定义规则来生成一系列元素填充新的数据集合。
这个概念我们在中学的数学课上其实就已经接触过，在大学高数中更为常见：
如`{ x | x ∈ N }`这个表达式，字面上意思是：这是一个集合，
这个集合里每个元素x符合“x属于自然数N”这个条件。即，用自然数集合的所有元素来构成这个集合。
相关知识可参考[WIKI](http://en.wikipedia.org/wiki/List_comprehension)。

Elixir中，使用枚举类型（Enumerable，如列表）来做循环操作是很常见的，
通常还搭配过滤（filtering）和映射（mapping）行为。
速构（comprehensions）就是为此目的诞生的语法糖：把这些常见任务分组，放到特殊的`for`指令中表达出来。

例如，我们可以这样，生成原列表中每个元素的平方：

```elixir
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```

>注意看，`<-`符号其实是模拟符号`∈ `的形象。
这个例子用熟悉（当然，如果你高数课没怎么听那就另当别论）的数学符号表示就是：

```
S = { X^2 | X ∈ [1,4], X ∈ N }
```

速构由三部分组成：生成器，过滤器和收集式。

## 生成器和过滤器

在上面的例子中，`n <- [1, 2, 3, 4]`就是生成器。
它字面意思上生成了即将要在速构中使用的数值。任何枚举类型（Enumerable）都可以传递给生成器表达式的右端：

```elixir
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```

生成器表达式左操作数支持模式匹配，它会**忽略**所有不匹配的模式。
想象一下如果不用范围而是用一个键值列表作为生成器的数据源，它的键只有`:good`和`:bad`两种，
我们仅要计算键为‘:good’的元素值的平方：

```elixir
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

除了使用模式匹配，过滤器也可以用来选择某些特定数值。
例如我们可以只选择3的倍数，而丢弃其它数值：

```elixir
iex> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
iex> for n <- 0..5, multiple_of_3?.(n), do: n * n
[0, 9]
```

速构过程会丢弃过滤器表达式结果为`false`或`nil`的值；其它值都会被保留。

总的来说，速构提供了比直接使用`Enum`或`Stream`模块的函数更精确的表达。
不但如此，速构还可以接受多个生成器和过滤器。下面就是一个例子，代码接受目录列表，
删除这些目录下的所有文件：

```elixir
for dir  <- dirs,
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  File.stat!(path).size
end
```

多生成器还可以用来生成两个列表的笛卡尔积：

```elixir
iex> for i <- [:a, :b, :c], j <- [1, 2], do:  {i, j}
[a: 1, a: 2, b: 1, b: 2, c: 1, c: 2]
```

关于多生成器、过滤器的更高级些的例子：计算毕达哥拉斯三元数（Pythagorean triples）。
毕氏三元数一组正整数满足`a * a + b * b = c * c`，让我们在文件`triples.exs`里写这个速构：

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n,
        b <- 1..n,
        c <- 1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

然后，在终端里：

```
iex triple.exs
```

```elixir
iex> Triple.pythagorean(5)
[]
iex> Triple.pythagorean(12)
[{3, 4, 5}, {4, 3, 5}]
iex> Triple.pythagorean(48)
[{3, 4, 5}, {4, 3, 5}, {5, 12, 13}, {6, 8, 10}, {8, 6, 10}, {8, 15, 17},
 {9, 12, 15}, {12, 5, 13}, {12, 9, 15}, {12, 16, 20}, {15, 8, 17}, {16, 12, 20}]
```

Finally, keep in mind that variable assignments inside the comprehension, be it in generators, filters or inside the block, are not reflected outside of the comprehension.

需要记住的是，在生成器、过滤器或者代码块中赋值的变量，不会暴露到速构外面去。

## 比特串生成器

速构也支持比特串作为生成器，这种生成器在处理比特流时非常有用。
下面的例子中，程序接收一个表示像素颜色的比特串（格式为<<像素1的R值，像素1的G值，像素1的B值，
像素2的R值，像素2的G...>>），把它转换为三元元组的列表：

```elixir
iex> pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>
iex> for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
[{213, 45, 132}, {64, 76, 32}, {76, 0, 0}, {234, 32, 15}]
```

比特串生成器可以和“普通的”枚举类型生成器混合使用，过滤器也是。

## `:into`选项

在上面的例子中，速构返回列表作为结果。
但是，通过使用```:into```选项，速构的结果可以插入到不同的数据结构中。
例如，你可以使用比特串生成器加上```:into```来轻松地移除字符串中的空格：

```elixir
iex> for <<c <- " hello world ">>, c != ?\s, into: "", do: <<c>>
"helloworld"
```

集合、图、其他字典类型都可以传递给`:into`选项。总的来说，`:into`接受任何实现了_Collectable_协议的数据结构。

`:into`选项一个常见的作用是，不用操作键，而改变图中元素的值：

```elixir
iex> for {key, val} <- %{"a" => 1, "b" => 2}, into: %{}, do: {key, val * val}
%{"a" => 1, "b" => 4}
```

再来一个使用流的例子。因为`IO`模块提供了流（既是Enumerable也是Collectable）。
你可以使用速构实现一个回声终端，让其返回任何输入的字母的大写形式：

```elixir
iex> stream = IO.stream(:stdio, :line)
iex> for line <- stream, into: stream do
...>   String.upcase(line) <> "\n"
...> end
```

现在在终端中输入任意字符串，你会看到同样的内容以大写形式被打印出来。
不幸的是，这个例子会让你的shell陷入到该速构代码中，只能用Ctrl+C两次来退出:-)。
