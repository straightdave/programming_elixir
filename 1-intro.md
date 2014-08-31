1-简介
======
Elixir，读作[ɪ'lɪksər]，意思是灵丹妙药、圣水，而它目前的logo就是蓝色水滴。<br/>
Elixir是建立在Erlang虚拟机上的一门函数式的、支持元编程的语言。<br/>
Elixir是一门动态语言，语法本质上来自Erlang，借鉴了ruby。（设计者Jose Valim本人就是Rails的核心工程师之一，资深ruby程序员）。<br/>
Elixir目的是将Erlang虚拟机换个面貌呈现给消费者（程序员），使得更多人可以利用Erlang语言以下能力：<br/>
  - 并发
  - 分布式
  - 故障容忍
  - 可以进行代码热升级，等等 <br/>
Elixir ~~最重要的~~ 目的是提高Erlang语法的优美程度，最好弄得像ruby一样。<br/>

截至2014年8月31日，1.0.0面世在即，目前的稳定版本是0.15.1。相信1.0.0发布后，Elixir将会得到更大更快的发展。

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
附上加环境变量的命令
```
$ export PATH="$PATH:/path/to/elixir/bin"
```
如果你十分激进，可以直接选择编译安装github上的master分支：
```
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```
如果测试无法通过，可在[repo](https://github.com/elixir-lang/elixir)的Issue里汇报。

### 安装Erlang




