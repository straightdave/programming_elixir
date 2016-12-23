15-结构体
=========

在之前的某章中，我们学习了图（Map）：

```elixir
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

结构体是基于图的一个扩展。它引入了默认值、编译期验证。

定义一个结构体，只需调用`defstruct/1`：

```elixir
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end

```

`defstruct/1`的参数（一个键值列表）定义了结构体的字段和默认值。结构体使用了定义它的模块的名字。
像上面这个例子，我们定义的结构体叫做`User`。

现在可以用类似创建图的语法来创建结构体`User`：

```elixir
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

结构体提供*编译期验证*，在代码在编译时会检查结构体内仅有之前定义的字段（而且一个字段也不少）：

```elixir
iex> %User{ oops: :field }
** (CompileError) iex:3: unknown key :oops for struct User
```

## 访问和修改结构体

当讨论图的时候，我们演示了如何访问和修改图的字段。
访问和修改结构体的技术（以及语法）也是一样的：

```elixir
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "Meg"}
```

当使用语法标记（`|`）的时候，虚拟机知道并没有增加新的字段，这使得底层的图可以共享内存中得结构。
像上文的例子，`john`和`meg`在内存中使用相同的键值结构。

结构体也能用在模式匹配中，这不但可以保证结构体的字段名称的值相同，也可以确保对应的字段值的类型也相同：

```elixir
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## 结构体和底层的图

在上面的例子里，之所以可以用模式匹配，是因为结构体底层不过是拥有固定字段的图。
而作为图，结构体还存储了一个名叫`__struct__`的特殊字段，来存储结构体的名字：

```elixir
iex> is_map(john)
true
iex> john.__struct__
User
```

简单说，结构体就是个被扒光的图外加一个默认字段。
为啥说是被扒光的图？因为，所有为图实现的协议（protocols）都不能用于结构体。
例如，你不能像对图那样枚举或直接访问一个结构体：

```elixir
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (UndefinedFunctionError) undefined function: User.fetch/2
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

尽管如此，因为结构体说到底还是图，对图有效的函数也可以作用于结构体：

```elixir
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

结构体和协议（protocols），为Elixir程序员提供了一个最重要的特性：数据多态。我们会在下一章解释和学习。

下一章我们将介绍结构体是如何同协议进行交互的。
