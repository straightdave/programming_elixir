13-别名和代码引用
=================
[别名]() <br/>
[require]() <br/>
[import]() <br/>
[系统别名]() <br/>
[嵌套]() <br/>

为了实现软件重用，Elixir提供了三种指令（directives）。之所以称之为“指令”是因为它们有词法上的可见范围（lexicla scope）。

## 13.1-别名
宏```alias```可以为任何模块名设置别名。想象一下Math模块，它针对特殊的数学运算使用了特殊的列表实现：
```
defmodule Math do
  alias Math.List, as: List
end
```

