10-Enumerables和流
==================
[Enumerables]() <br/>
[积极vs懒惰]() <br/>
[流]() <br/>

## 10.1-Enumerables
Elixir提供了enumerables的概念，使用[Enum模块]()操作它们。我们已经介绍过两种enumerables：列表和图。
```
iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]
```

Enum模块为enumerables类型提供了大量函数来变化，排序，分组，过滤和读取元素。
