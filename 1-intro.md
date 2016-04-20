1-简介
======

欢迎！   

>
本章是主要讲了各个平台上如何安装使用Elixir。由于本文主要关注Elixir的语言学习，
因此这个章节所讲的步骤或工具可能不是最新，请大家自行网上搜索。

本章将涵盖如何安装Elixir，并且学习使用交互式的Elixir Shell（称为IEx）。

使用本教程的需求：
  - Erlang - version 17.0 或更高
  - Elixir - 1.0.0 或更高

现在开始吧！

>如果你发现本手册有错误，请帮忙开_issue_讨论或发_pull request_。

## 1.1-安装包
在各个平台上最方便的安装方式是相应平台的安装包。
如果没有，推荐使用precompiled package或者用源码编译安装。   

注意Elixir需要Erlang 17.0或更高。下面介绍的方法基本上都会自动为你安装Erlang。
假如没有，请阅读下面安装Erlang的说明。

### Mac OS X
- Homebrew
  - 升级Homebrew到最新
  - 执行：```brew install elixir```
- Macports
  - 执行：```sudo port install elixir```

### Unix（或者类Unix）
  - Fedora 17或更新
    - 执行：```yum install elixir```
  - Fedora 22或更新
    - 执行：```dnf install elixir```
  - Arch Linux (社区repo)
    - 执行：```pacman -S elixir```
  - openSUSE (and SLES 11 SP3+)
    - 添加Erlang devel repo: ```zypper ar -f obs://devel:languages:erlang/ erlang```
    - 执行：```zypper in elixir```
  - Gentoo
    - 执行：```emerge --ask dev-lang/elixir```
  - FreeBSD
    - 使用ports: ```cd /usr/ports/lang/elixir && make install clean```
    - 或使用pkg: ```pkg install elixir```
  - Ubuntu 12.04和14.04，或Debian 7
    - 添加Erlang Solutions repo: ```wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb && sudo dpkg -i erlang-solutions_1.0_all.deb```
    - 执行：```sudo apt-get update```
    - 安装Erlang/OTP平台及相关程序：```sudo apt-get install esl-erlang```
    - 安装Elixir：```sudo apt-get install elixir```

### Windows
  - Web installer
    - [下载installer](https://s3.amazonaws.com/s3.hex.pm/elixir-websetup.exe)
    - 点下一步，下一步...直到完成
  - Chocolatey
    - ```cinst elixir ```

## 1.3-使用预编译包
Elixir为每一个release提供了预编译包（编译好并打包的程序，开箱即用）。   
首先[安装Erlang](http://elixir-lang.org/install.html#installing-erlang)，
然后在[这里](https://github.com/elixir-lang/elixir/releases/)下载最新的
预编译包（Precompiled.zip）。unzip，即可使用elixir程序和iex程序了。   
当然为了方便起见，需要将一些路径加入环境变量。

## 1.4-从源码编译安装（Unix和MinGW）
首先[安装Erlang](http://elixir-lang.org/install.html#installing-erlang)，
然后在[这里](https://github.com/elixir-lang/elixir/releases/)下载最新的源码，
自己使用make工具编译安装。

>在Windows上编译安装请参考https://github.com/elixir-lang/elixir/wiki/Windows

>附上加环境变量的命令
```sh
$ export PATH="$PATH:/path/to/elixir/bin"
```

如果你十分激进，可以直接选择编译安装github上的master分支：
```sh
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```
如果测试无法通过，可在[repo](https://github.com/elixir-lang/elixir)里开issue汇报。

## 1.5-安装Erlang
安装Elixir唯一的要求就是Erlang（V17.0+），
它可以很容易地使用
[预编译包](https://www.erlang-solutions.com/downloads/download-erlang-otp)安装。
如果你想从源码安装，可以去[Erlang网站](http://www.erlang.org/download.html)找找，
参考[Riak文档](http://docs.basho.com/riak/1.3.0/tutorials/installation/Installing-Erlang/)。   
安装好Erlang后，打开命令行（或命令窗口），输入```erl```，可以输出Erlang的版本信息：
```
Erlang/OTP 17 (erts-6) [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]
```
>安装好Erlang后，你需要手动添加环境变量或$PATH。
关于环境变量，参考[这里](http://en.wikipedia.org/wiki/Environment_variable)。


## 1.6-交互模式
安装好Elixir之后，你有了三个可执行文件：```iex```，```elixir```和```elixirc```。
如果你是用预编译包方式安装的，可以在解压后的bin目录下找到它们。    

现在我们可以从```iex```开始了（或者是```iex.bat```，如果在Windows上）。
交互模式，就是可以向其中输入任何Elixir表达式或命令，然后直接看到表达式或命令的结果。
如以下所示：
```elixir
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)

iex> 40 + 2
42
iex> "hello" <> " world"
"hello world"
```
对这种交互式命令行，相信熟悉ruby，python等动态语言的程序员一定不会陌生。

>如果你用的是Windows，你可以使用```iex.bat --werl```，可以根据你的console获得更好的使用体验。

## 1.7-执行脚本
把表达式写进脚本文件，可以用```elixir```命令执行它。如：
```sh
$ cat simple.exs
IO.puts "Hello world
from Elixir"

$ elixir simple.exs
Hello world
from Elixir
```

在以后的章节中，我们还会介绍如何编译Elixir程序，以及使用Mix这样的构建工具。
