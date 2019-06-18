---
layout: post
title:  "Polymorphism in Elixir"
date:   2019-06-17
---

While we tend to associate polymorphism with object-oriented design, the functional language, Elixir, allows us to create duck types through a mechanism called [protocols](https://elixir-lang.org/getting-started/protocols.html).

Let's imagine we have an Elixir game program that has a board of tiles. A tile can either be a `BombTile` struct or an `EmptyTile` struct.

{% highlight elixir %}
defmodule BombTile do
  defstruct exploded: false
end

defmodule EmptyTile do
  defstruct revealed: false
end
{% endhighlight %}

Each of these structs has a different property: `:exploded` and `:revealed`, which are `false` by default. When a user clicks on a tile, we expect it to either explode if it's a bomb, or be revealed if it's empty.

In order to change the `:exploded` and `:revealed` properties on the tiles to `true` , we'll add a `select/1` function to each module.

{% highlight elixir %}
defmodule BombTile do
  defstruct exploded: false

  def select(bomb_tile) do
    %{ bomb_tile | exploded: true }
  end
end

defmodule EmptyTile do
  defstruct revealed: false

  def select(empty_tile) do
    %{ empty_tile | revealed: true }
  end
end
{% endhighlight %}

Now, let's add a `Board` module that contains a function to "select" a tile at a given index on a board (because everything is immutable in Elixir, this function is going to return a new list of tiles). 

{% highlight elixir %}
defmodule Board do
  def select_tile(board, index) do
    tile = Enum.at(board, index)

    selected_tile =
      cond do
        tile.__struct__ == BombTile ->
          BombTile.select(tile)

        tile.__struct__ == EmptyTile ->
          EmptyTile.select(tile)
      end

    List.replace_at(board, index, selected_tile)
  end
end
{% endhighlight %}

Because we haven’t implemented polymorphism yet, we have to check the struct of the tile in order to know which module to send the `select/1` message to. Now let's see this code in action.

```
iex> board = [%BombTile{}, %EmptyTile{}]
[
  %BombTile{exploded: false},
  %EmptyTile{revealed: false}
]

iex> Board.select_tile(board, 0)
[
  %BombTile{exploded: true},
  %EmptyTile{revealed: false}
]

iex> Board.select_tile(board, 1)
[
  %BombTile{exploded: false},
  %EmptyTile{revealed: true}
]
```

It works as expected, but this antipattern is reminiscent of the one we discovered in our discussion of [duck typing in object-oriented design]({{ site.baseurl }}{% post_url 2019-03-21-design-techniques-duck-typing %}). Even in a functional language, checking the type of a data structure before deciding which message to send is a code smell. Let's fix this by implementing a `Tile` protocol.

{% highlight elixir %}
defprotocol Tile do
  def select(tile)
end
{% endhighlight %}

This is the protocol definition. Any module that implements a protocol needs to implement the functions defined in said protocol. Now let's implement the `Tile` protocol for our `BombTile` and `EmptyTile` modules. 

{% highlight elixir %}
defimpl Tile, for: BombTile do
  def select(bomb_tile) do
    %{ bomb_tile | exploded: true }
  end
end

defimpl Tile, for: EmptyTile do
  def select(empty_tile) do
    %{ empty_tile | revealed: true }
  end
end
{% endhighlight %}

The first argument of each function has to be a data type that implements the protocol. Also, remember to remove the original `select/1` functions from `BombTile` and `EmptyTile` as they're no longer needed there.

Now, let's refactor our `Board.select/2` function to use the `Tile` protocol.

{% highlight elixir %}
defmodule Board do
  def select_tile(board, index) do
    List.replace_at(board, index, Tile.select(tile))
  end
end
{% endhighlight %}

And now you know how to achieve polymorphism in Elixir!
