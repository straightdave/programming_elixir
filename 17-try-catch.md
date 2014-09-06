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
