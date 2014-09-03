9-循环
======
因为在Elixir中（或所有函数式语言中），数据有不变性（immutability），因此在写循环时与传统的命令式（imperative）语言有所不同。
例如某命令式语言的循环可以这么写：
```
for(i = 0; i < array.length; i++) {
  array[i] = array[i] * 2
}
```

上面例子中，我们改变了```array```，以及辅助变量```i```的值。这在Elixir中是不可能的。

