6-二进制-字符串-字符列表
========================
[UTF-8和Unicode]() <br/>
[二进制（和bitstring）]()<br/>
[字符列表]() <br/>

在“基本类型”一章中，介绍了字符串，以及用is_binary/1函数检查它：
```
iex> string = "hello"
"hello"
iex> is_binary string
true
```

本章将学习理解，二进制（binaries）是个啥，它怎么和字符串扯上关系的，以及用单引号包裹的值，```'like this'```，是啥意思。

## 6.1-UTF-8和Unicode
