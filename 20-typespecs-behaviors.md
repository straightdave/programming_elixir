20-Typespecs和behaviors
=======================

## Types and specs

Elixir是一门动态类型语言，Elixir中所有类型都是在运行时动态推定的。
然而，Elixir还是提供了 **typespecs** 标记，用来：   
1. 声明自定义数据类型
2. 声明含有类型的函数签名（即函数的规格说明）

### 函数的规格说明

默认地，Elixir提供了一些基础类型，比如`integer`或者`pid`，还有其它一些复杂的：
如`round/1`函数，它对一个float类型四舍五入。它以一个`number`作为参数
（一个`integer`或`float`），返回一个`integer`。   
你可以阅读[这篇文档](http://elixir-lang.org/docs/stable/elixir/Kernel.html#round/1),
`round/1`含有类型的签名为：
```
round(number) :: integer
```

`::` means that the function on the left side *returns* a value whose type is what's on the right side. Function specs are written with the `@spec` directive, placed right before the function definition. The `round/1` function could be written as:

```elixir
@spec round(number) :: integer
def round(number), do: # implementation...
```

Elixir supports compound types as well. For example, a list of integers has type `[integer]`. You can see all the built-in types provided by Elixir [in the typespecs docs](/docs/stable/elixir/typespecs.html).

### Defining custom types

While Elixir provides a lot of useful built-in types, it's convenient to define custom types when appropriate. This can be done when defining modules through the `@type` directive.

Say we have a `LousyCalculator` module, which performs the usual arithmetic operations (sum, product and so on) but, instead of returning numbers, it returns tuples with the result of an operation as the first element and a random remark as the second element.

```elixir
defmodule LousyCalculator do
  @spec add(number, number) :: {number, String.t}
  def add(x, y), do: {x + y, "You need a calculator to do that?!"}

  @spec multiply(number, number) :: {number, String.t}
  def multiply(x, y), do: {x * y, "Jeez, come on!"}
end
```

As you can see in the example, tuples are a compound type and each tuple is identified by the types inside it. To understand why `String.t` is not written as `string`, have another look at the [notes in the typespecs docs](/docs/stable/elixir/typespecs.html#Notes).

Defining function specs this way works, but it quickly becomes annoying since we're repeating the type `{number, String.t}` over and over. We can use the `@type` directive in order to declare our own custom type.

```elixir
defmodule LousyCalculator do
  @typedoc """
  Just a number followed by a string.
  """
  @type number_with_remark :: {number, String.t}

  @spec add(number, number) :: number_with_remark
  def add(x, y), do: {x + y, "You need a calculator to do that?"}

  @spec multiply(number, number) :: number_with_remark
  def multiply(x, y), do: {x * y, "It is like addition on steroids."}
end
```

The `@typedoc` directive, similarly to the `@doc` and `@moduledoc` directives, is used to document custom types.

Custom types defined through `@type` are exported and available outside the module they're defined in:

```elixir
defmodule QuietCalculator do
  @spec add(number, number) :: number
  def add(x, y), do: make_quiet(LousyCalculator.add(x, y))

  @spec make_quiet(LousyCalculator.number_with_remark) :: number
  defp make_quiet({num, _remark}), do: num
end
```

If you want to keep a custom type private, you can use the `@typep` directive instead of `@type`.

### Static code analysis

Typespecs are not only useful to developers and as additional documentation. The Erlang tool [Dialyzer](http://www.erlang.org/doc/man/dialyzer.html), for example, uses typespecs in order to perform static analysis of code. That's why, in the `QuietCalculator` example, we wrote a spec for the `make_quiet/1` function even if it was defined as a private function.

## Behaviours

Many modules share the same public API. Take a look at [Plug](https://github.com/elixir-lang/plug), which, as its description states, is a **specification** for composable modules in web applications. Each *plug* is a module which **has to** implement at least two public functions: `init/1` and `call/2`.

Behaviours provide a way to:

* define a set of functions that have to be implemented by a module;
* ensure that a module implements all the functions in that set.

If you have to, you can think of behaviours like interfaces in object oriented languages like Java: a set of function signatures that a module has to implement.

### Defining behaviours

Say we want to implement a bunch of parsers, each parsing structured data: for example, a JSON parser and a YAML parser. Each of these two parsers will *behave* the same way: both will provide a `parse/1` function and an `extensions/0` function. The `parse/1` function will return an Elixir representation of the structured data, while the `extensions/0` function will return a list of file extensions that can be used for each type of data (e.g., `.json` for JSON files).

We can create a `Parser` behaviour:

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end
```

Modules adopting the `Parser` behaviour will have to implement all the functions defined with the `@callback` directive. As you can see, `@callback` expects a function name but also a function specification like the ones used with the `@spec` directive we saw above.

### Adopting behaviours

Adopting a behaviour is straightforward:

```elixir
defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```

```elixir
defmodule YAMLParser do
  @behaviour Parser

  def parse(str), do: # ... parse YAML
  def extensions, do: ["yml"]
end
```

If a module adopting a given behaviour doesn't implement one of the callbacks required by that behaviour, a compile-time warning will be generated.
