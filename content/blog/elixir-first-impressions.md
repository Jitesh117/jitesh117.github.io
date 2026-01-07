+++
title = 'My first impressions on elixir'
date = 2026-01-07T21:46:21+05:30
draft = false
showToc = true
cover.image = 'images/elixir.png'
+++

Some days back, I started learning Elixir. And boy oh boy, learning Elixir is an experience in itself. There were a number of peculiarities I found while going through this functional programming language.

Also, for the record, I am following the book [Elixir in Action](https://www.manning.com/books/elixir-in-action-third-edition) to learn this beautiful language.

Right off the bat, these were the things I noticed to be different than any language I have ever worked with:

## 1. Inspired by Ruby and Lisp

You can clearly see Ruby’s influence in the syntax, and Lisp’s influence in the functional ideas. It feels familiar, but also very different at the same time.

## 2. No return statements

There are _no_ return statements in Elixir. Yes, you heard it right. What it actually does is treat the last expression as the return value.

For example:

```elixir
defmodule Geometry do
  @type sides :: pos_integer()
  @type length :: number()
  @type area :: float()

  @type error_reason ::
          :invalid_sides
          | :invalid_length

  @type result :: {:ok, area()} | {:error, error_reason()}

  @spec polygon_area(sides(), length()) :: result()

  # Branch 1: invalid number of sides
  def polygon_area(sides, _length) when sides < 3 do
    {:error, :invalid_sides}
  end

  # Branch 2: invalid side length
  def polygon_area(_sides, length) when length <= 0 do
    {:error, :invalid_length}
  end

  # Branch 3: valid polygon
  def polygon_area(sides, length) do
    area =
      sides * length * length /
        (4 * :math.tan(:math.pi() / sides))

    {:ok, area}
  end
end
```

Here, you can see how the function returns different values based on certain conditions, without a single `return` keyword.

## 3. No for / while loops

Yup. Elixir does not rely on `for` or `while` loops the way most languages do. Instead, it heavily uses recursion for iteration.

You might think this would blow up the call stack. But Elixir handles this with something called _tail call optimization_. When the last thing a function does is call itself, it reuses the stack frame instead of creating a new one.

For example, summing a list:

```elixir
defmodule ListHelper do
  def sum(list) do
    do_sum(0, list)
  end

  defp do_sum(current_sum, []) do
    current_sum
  end

  defp do_sum(current_sum, [head | tail]) do
    new_sum = head + current_sum
    do_sum(new_sum, tail)
  end
end
```

The `defp` keyword defines private functions. Or rather, a macro. Which brings me to the next point.

## 4. A lot of things are macros

Many things that look like keywords are actually macros built on top of Elixir itself. This includes `def`, `defp`, and more.

This makes the language very flexible and powerful, but also a bit mind-bending when you are new to it.

## 5. Data is immutable

You cannot change data in Elixir. You can only create new data.

Even the `=` operator is not an assignment operator. It is used for _pattern matching_.

So when you write:

```elixir
a = 1
a = 2
```

You are not changing the value of `a`. You are rebinding it to a new value. The old value still exists somewhere in memory until the garbage collector cleans it up.

## 6. Pattern matching is everywhere

Pattern matching is not just a feature. It is everywhere.

Function arguments, case statements, variable binding, and more. Once you get used to it, it feels very natural and very powerful.

## 7. The pipe operator is super handy

The pipe operator lets you chain function calls in a clean way.

```elixir
-5
|> abs()
|> Integer.to_string()
|> IO.puts() # prints "5"
```

This keeps code readable and easy to follow.

## 8. Everything runs in processes

Elixir uses lightweight processes, not OS threads. These processes are cheap, isolated, and communicate only by sending messages.

This makes concurrent programming much simpler and safer compared to shared-memory models.

## 9. Crashing is okay

Elixir follows the “let it crash” philosophy. Instead of trying to handle every error everywhere, you let processes fail and let supervisors restart them.

This leads to systems that are easier to reason about and more reliable in the long run.

## 10. Error handling is explicit

Most functions return tuples like `{:ok, result}` or `{:error, reason}`.

You are forced to handle both cases, usually with pattern matching:

```elixir
case File.read("data.txt") do
  {:ok, content} -> IO.puts(content)
  {:error, reason} -> IO.inspect(reason)
end
```

No surprises. No hidden behavior.

## 11. Tooling is great

The `mix` tool makes project management simple.

Creating a project:

```bash
mix new my_app
```

Running tests:

```bash
mix test
```

Everything just works.

## 12. It forces you to think differently

Elixir does not let you write sloppy code. Immutability and explicit data handling push you to be clear about what your code is doing.

At first, it feels uncomfortable. Later, it feels safe.

## Final thoughts

Elixir is not easy if you are coming from an object-oriented background. It challenges your habits at every step.

But if you stick with it, it rewards you with clean code, strong guarantees, and a fresh way of thinking.

I am still very early in my Elixir journey, but so far, it has been a fun and humbling experience. And it has been kind of growing on me. I am excited to see where this goes.
