1-简介
======
Elixir，读作[ɪ'lɪksər]，意思是灵丹妙药、圣水，而它目前的logo就是蓝色水滴。<br/>
Elixir是一门建立在Erlang虚拟机上的*函数式语言*,支持元编程。<br/>
Elixir是一门动态语言，语法本质上来自Erlang，借鉴了ruby。（设计者Jose Valim本人就是Rails的核心工程师之一，资深ruby程序员）。<br/>
Elixir目的是将Erlang虚拟机换个面貌呈现给消费者（程序员），使得更多人可以利用Erlang语言以下能力：<br/>
  - 高并发
  - 分布式
  - 故障容忍
  - 可以进行代码热升级，等等 <br/>
Elixir ~~根本~~ 目的是提高Erlang语法的优美程度，最好弄得像ruby一样。<br/>

截至2014年8月31日，1.0.0面世在即，目前的稳定版本是0.15.1。相信1.0.0正式发布后，Elixir将会得到更大更快的发展。

[安装并且进一步了解Elixir](http://elixir-lang.org/getting_started/1.html) <br/>
[在线文档](http://elixir-lang.org/docs.html) <br/>
[故障报告](http://elixir-lang.org/crash-course.html) <br/>

## 1.1-安装
以下讲解了如何安装Elixir并且使用它的交互式Shell（IEx - Interactive Elixir）。
（Erlang V17.0+，Elixir V0.15.0+）

### 自动安装
Windows Installer：[Here](http://s3.hex.pm/elixir-websetup.exe) <br/>
该Installer包括了最新版本的Elixir和Erlang。

其它平台：<br/>
  - 在MaxOS X上使用Homebrew
    - brew update
    - brew install elixir
  - 在MacOS X上使用Macports
    - sudo port install elixir
  - Fedora 17+/Rawhide
    - sudo yum -y install elixir
  - Arch Linux (on AUR)
    - yaourt -S elixir
  - openSUSE (and SLES 11 SP3+)
    - ar -f obs://devel:languages:erlang/ erlang
    - zypper in elixir
  - Gentoo
    - emerge --ask dev-lang/elixir
  - 在Windows上使用Chocolatey
    - cinst elixir
  - FreeBSD
    - 使用ports: cd /usr/ports/lang/elixir && make install clean
    - 或使用pkg: pkg install elixir
<br/>
以上方法都应该会自动安装Erlang，如果没有，请参考[下面章节](https://github.com/straightdave/programming-elixir/blob/master/1-intro.md#%E5%AE%89%E8%A3%85erlang)。<br/>

### 使用预编译包
如果想尝鲜，Elixir为每一个release提供了预编译包（编译好并打包的程序，开箱即用）。
首先[安装Erlang](http://elixir-lang.org/getting_started/1.html#1.5-installing-erlang)，然后在[这里](https://github.com/elixir-lang/elixir/releases/)下载最新的预编译包（Precompiled.zip），开zip，即可使用elixir和iex了。
当然为了方便起见，可将这些可执行文件的路径加入环境变量。

### 自己编译安装
首先[安装Erlang](http://elixir-lang.org/getting_started/1.html#1.5-installing-erlang)，然后在[这里](https://github.com/elixir-lang/elixir/releases/)下载最新的源码，自己使用make工具编译安装。
注意：在Windows上编译安装请参考https://github.com/elixir-lang/elixir/wiki/Windows
- 附上加环境变量的命令
```
$ export PATH="$PATH:/path/to/elixir/bin"
```
- 如果你十分激进，可以直接选择编译安装github上的master分支：
```
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```
如果测试无法通过，可在[repo](https://github.com/elixir-lang/elixir)的Issue里汇报。

### 安装Erlang
安装Elixir唯一的要求就是Erlang（V17.0+），它可以很容易地使用[预编译包](https://www.erlang-solutions.com/downloads/download-erlang-otp)安装。如果你想从源码安装，可以去[Erlang网站](http://www.erlang.org/download.html)找找，参考[Riak文档](http://docs.basho.com/riak/1.3.0/tutorials/installation/Installing-Erlang/)。<br/>
安装好Erlang后，打开命令行（或命令窗口），输入```erl```，应该可以输出Erlang的版本信息，例如：
```
Erlang/OTP 17 (erts-6) [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]
```
注意：安装好Erlang后，你需要手动添加环境变量或$PATH。关于环境变量，参考[这里](http://en.wikipedia.org/wiki/Environment_variable)。
<br/>

## 1.2 交互模式
安装好Elixir之后，你有了三个可执行文件：```iex```，```elixir```和```elixirc```。
如果你是用预编译包方式安装的，可以在解压后的bin目录下找到它们。  
现在我们可以从```iex```开始了（或者是```iex.bat```，如果在Windows上）。交互模式，就是可以向其中输入任何Elixir表达式或命令，然后直接看到表达式或命令的结果。如以下简单的几条命令：
```
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)

iex> 40 + 2
42
iex> "hello" <> " world"
"hello world"
```
对这种交互式命令行，相信熟悉ruby，PHP等动态语言的程序员一定不会陌生。

## 1.3 执行脚本
把表达式写进脚本文件，可以用```elixir```命令执行它。如：
```
$ cat simple.exs
IO.puts "Hello world
from Elixir"

$ elixir simple.exs
Hello world
from Elixir
```
在以后的章节中，我们还会介绍如何编译Elixir程序，以及Mix这样的build工具。




