17-异常处理
===========
[Errors]()<br/>
[Throws]()<br/>
[Exits]()<br/>
[After]()<br/>
[变量作用域]()<br/>

Elixir有三种错误机制：errors，throws和exits。本章我们将逐个讲解它们，包括在何时使用哪一个。

## 17.1-Errors
举个例子，尝试让原子加上一个数字，就会激发一个错误（errors）：
```
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

使用宏```raise/1```可以在任何时候激发一个运行时错误：
```
iex> raise "oops"
** (RuntimeError) oops
```

用```raise/2```，并且附上错误名称和一个键值列表可以激发规定好的错误：
```
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

你好可以使用```defexception/2```定义你自己的错误。最常见的是定义一个有消息说明的错误：
```
iex> defexception MyError, message: "default message"
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

用```try/catch```结构可以处理异常：
```
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
RuntimeError[message: "oops"]
```
这个例子处理了一个运行时异常，返回该错误本身（会被显示在IEx中）。























