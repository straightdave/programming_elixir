15-结构体
=========
在之前的几章中，我们谈到过图：
```
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

结构体是基于图的一个扩展。它将默认值，编译时保证和多态性带入Elixir。

定义一个结构体，你只需在某模块中调用```defstruct/1```：
```
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
 ```
 
 现在可以用```%User()```语法创建这个结构体的“实例”了：
 ```
iex> %User{}
%User{name: "john", age: 27}
iex> %User{name: "meg"}
%User{name: "meg", age: 27}
iex> is_map(%User{})
true
```

结构体的编译时保证指的是编译时会检查结构体的字段存不存在：
```
iex> %User{oops: :field}
** (CompileError) iex:3: unknown key :oops for struct User
```

当讨论图的时候，我们演示了如何访问和修改图现有的字段。结构体也是一样的：
```
iex> john = %User{}
%User{name: "john", age: 27}
iex> john.name
"john"
iex> meg = %{john | name: "meg"}
%User{name: "meg", age: 27}
iex> %{meg | oops: :field}
** (ArgumentError) argument error
```

通过使用这种修改的语法，虚拟机知道没有新的键增加到图/结构体中，使得图可以在内存中共享它们的结构。
在上面例子中，john和meg共享了相同的键结构。

结构体也能用在模式匹配中，它们保证结构体有相同的类型：
```
iex> %User{name: name} = john
%User{name: "john", age: 27}
iex> name
"john"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

匹配能工作，是由于结构体再底层图中有个字段叫```__struct__```：
```
iex> john.__struct__
User
```

总之，结构体就是个光秃秃的图外加一个默认字段。我们这里说的光秃秃的图指的是，为图实现的协议都不能用于结构体。
例如，你不能枚举也不能访问一个结构体：
```
iex> user = %User{}
%User{name: "john", age: 27}
iex> user[:name]
** (Protocol.UndefinedError) protocol Access not implemented for %User{age: 27, name: "john"}
```

结构体也不是字典，因为也不适用字典模块的函数：
```
iex> Dict.get(%User{}, :name)
** (ArgumentError) unsupported dict: %User{name: "john", age: 27}
```

下一章我们将介绍结构体是如何同协议进行交互的。
