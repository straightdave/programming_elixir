20-下一步
==========

还想学习更多？继续阅读吧！

## 构建第一个Elixir工程

Elixir提供了一个构建工具叫做Mix。你可以简单地执行以下命令来开始构建项目：

```
mix new path/to/new/project
```

这里已经写好了一份手册，讲述了如何构建一个Elixir应用程序，包括它自己的监督树，配置，测试等等。
这个应用程序是一个分布式的键-值对存储程序。它以键-值对的形式将数据存储在“桶”里，
并且把这些“桶”分发到不同的节点：

  - [Advanced Elixir](https://github.com/straightdave/advanced_elixir)

## 元编程

感谢语言上的元编程支持，Elixir具有可扩展性和高度定制性。
Elixir的元编程主要是由“宏”实现。在许多情景，特别是开发DSL特别有用。
这里也有一份手册文档，讲述了宏背后的基础原理，以及如何利用宏实现元编程来创建DSL：

  - [Elixir元编程](https://github.com/straightdave/elixir_meta_programming)

## 社区和其它资源

社区内的[“学习”部分](http://elixir-lang.org/learning.html)
推荐了一些书籍、视频等资源来学习Elixir，探索其生态系统。
还有一些，如程序大会的演讲、开源项目等由社区创建的学习资料在那里。

记住如果遇到困难，可以访问 __#elixir-lang__ 频道（__irc.freenode.net__），
或是向邮件列表中发信。可以肯定那里会有人愿意提供帮助。收藏博客或是
[订阅邮件列表](https://groups.google.com/group/elixir-lang-core)
以接收最新的新闻和声明。

别忘记还可以阅读Elixir的源码，其大部分使用Elixir写的（主要是lib那个目录下）。
或是[阅读Elixir的文档](http://elixir-lang.org/docs.html)。

## 来点Erlang

Elixir运行于Erlang虚拟机。不久之后，Elixir的开发者会完成对所有Erlang库的连接。
以下这些在线资源包含了Erlang的基础知识及高级特性：

  - [Erlang语法速成](http://elixir-lang.org/crash-course.html)
  提供了Erlang语法的简要介绍。每个代码片段都附有对等的Elixir代码。
  这不但有助于你一窥Erlang的语法，还可以复习你在本指导书里学到的知识。

  - Erlang的官方站点有份简短的
  [图文教程](http://www.erlang.org/course/concurrent_programming.html)
  简要描述了Erlang并行编程的原语。

  - [为你好学点Erlang吧](http://learnyousomeerlang.com/)
  是极好的Erlang介绍：它的设计原则、标准库、最佳实践等等等等。
  只要阅读上述速成，你就可以安全地忽略一些Erlang教课书前面介绍基础语法的几章。当阅读到
  [并发指南](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency)
  时，真正的愉悦开始了。
