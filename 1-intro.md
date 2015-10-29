1-简介
======
[安装程序](#11-%E5%AE%89%E8%A3%85%E7%A8%8B%E5%BA%8F) <br/>
[其它平台](#12-%E5%85%B6%E5%AE%83%E5%B9%B3%E5%8F%B0) <br/>
[使用预编译包](#13-%E4%BD%BF%E7%94%A8%E9%A2%84%E7%BC%96%E8%AF%91%E5%8C%85) <br/>
[从源码编译安装](#14-%E4%BB%8E%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85) <br/>
[安装Erlang](#15-%E5%AE%89%E8%A3%85erlang) <br/>
[交互模式](#16-%E4%BA%A4%E4%BA%92%E6%A8%A1%E5%BC%8F) <br/>
[执行脚本](#17-%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC) <br/>

欢迎！
>本文属于Elixir语法入门教程。推荐边看入门，边看Erlang/OTP的资料。因为Elixir只是一门编程语言，精华是在用其于开发基于OTP的应用。   
Elixir是为了改进Erlang晦涩的语法，使之变得像Ruby那么美观，让人们可以更方便地利用OTP开发高度稳定和容错的应用。

本章将涵盖如何安装，如何学习使用交互式Elixir Shell（称为IEx）。

使用本教程的需求：
  - Erlang - V17.0或更高
  - Elixir - V0.15.0或更高
<br/>
开始吧！

## 1.1-安装程序
Elixir为Windows平台提供了安装程序（Installer）：
  - Windows Installer：[Here](http://s3.hex.pm/elixir-websetup.exe) 
<br/>
该安装程序包括了最新版本的Elixir和Erlang。

## 1.2-其它平台
Elixir可以工作在以下系统平台上：
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
以上方法都应该会自动安装Erlang.如果没有，请参考[1.5-安装Erlang](#15-%E5%AE%89%E8%A3%85erlang)。<br/>

Ubuntu用户：
  - 最方便的方法就是安装Erlang后，下载与编译包。解压并且export PATH。

## 1.3-使用预编译包
如果想尝鲜，Elixir为每一个release提供了预编译包（编译好并打包的程序，开箱即用）。<br/>
首先[安装Erlang](http://elixir-lang.org/getting_started/1.html#1.5-installing-erlang)，然后在[这里](https://github.com/elixir-lang/elixir/releases/)下载最新的预编译包（Precompiled.zip），开zip，即可使用elixir和iex了。<br/>
当然为了方便起见，可将这些可执行文件的路径加入环境变量。

## 1.4-从源码编译安装
首先[安装Erlang](http://elixir-lang.org/getting_started/1.html#1.5-installing-erlang)，
然后在[这里](https://github.com/elixir-lang/elixir/releases/)下载最新的源码，自己使用make工具编译安装。

>在Windows上编译安装请参考https://github.com/elixir-lang/elixir/wiki/Windows

>附上加环境变量的命令
```
$ export PATH="$PATH:/path/to/elixir/bin"
```

>如果你十分激进，可以直接选择编译安装github上的master分支：
```
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```
如果测试无法通过，可在[repo](https://github.com/elixir-lang/elixir)的Issue里汇报。

## 1.5-安装Erlang
安装Elixir唯一的要求就是Erlang（V17.0+），它可以很容易地使用[预编译包](https://www.erlang-solutions.com/downloads/download-erlang-otp)安装。
如果你想从源码安装，可以去[Erlang网站](http://www.erlang.org/download.html)找找，参考[Riak文档](http://docs.basho.com/riak/1.3.0/tutorials/installation/Installing-Erlang/)。<br/>
安装好Erlang后，打开命令行（或命令窗口），输入```erl```，可以输出Erlang的版本信息：
```
Erlang/OTP 17 (erts-6) [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]
```
>安装好Erlang后，你需要手动添加环境变量或$PATH。关于环境变量，参考[这里](http://en.wikipedia.org/wiki/Environment_variable)。


## 1.6-交互模式
安装好Elixir之后，你有了三个可执行文件：```iex```，```elixir```和```elixirc```。
如果你是用预编译包方式安装的，可以在解压后的bin目录下找到它们。  <br/>
现在我们可以从```iex```开始了（或者是```iex.bat```，如果在Windows上）。
交互模式，就是可以向其中输入任何Elixir表达式或命令，然后直接看到表达式或命令的结果。
如以下所示：
```
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)

iex> 40 + 2
42
iex> "hello" <> " world"
"hello world"
```
对这种交互式命令行，相信熟悉ruby，python等动态语言的程序员一定不会陌生。

## 1.7-执行脚本
把表达式写进脚本文件，可以用```elixir```命令执行它。如：
```
$ cat simple.exs
IO.puts "Hello world
from Elixir"

$ elixir simple.exs
Hello world
from Elixir
```

在以后的章节中，我们还会介绍如何编译Elixir程序，以及使用Mix这样的build工具。
