---
layout: post
title:  "SOLID: Open-Closed Principle"
date:   2019-04-22
categories: ruby
---

The authors of _Agile Software Development: Principles, Patterns, and Practices_ (PPP) claim that the Open-Closed Principle (OCP) is at the heart of object-oriented design. It is this principle that gives our code the most flexibility, maintainability, and reusability, and it prioritizes adding new code over changing old code that already works.

The OCP is composed of two ideas—your code should be both open for extension _and_ closed for modification. A module that is closed for modification will not incur changes to its source code, and being open for extension means that the same module can still gain new behaviour. At first these two ideas seem mutually exclusive—how can we change a module without changing its code? The key is abstraction, and the strategy pattern in particular is a way of applying the OCP. Let’s look at an example.

In my Ruby implementation of Battleship, I expect the players to place a move on a board during their respective turns. In my game, I have both human players and computer players, and while I want the human players to supply the location of their next move via the user interface, I want the computer players to randomly generate a location. Here’s how you might implement this feature without the OCP.

{% highlight ruby %}
class Player
  attr_reader :type

  def initialize(type:)
    @type = type
  end

  def take_turn(board:)
    location = case type
    when :human
      get_human_move
    when :computer
      get_computer_move
    end
    board.get_tile(location).destroy
  end

  def get_human_move
    # ask for a move via the user interface
  end

  def get_computer_move
    # randomly generate a move
  end
end
{% endhighlight %}

What would we have to do in order to _extend_ this piece of code? Let’s say we wanted to add a `:smart_computer` player type that, instead of randomly selecting a move, would try to figure out where an opponent's ship was likely to be. Making this change would involve adding another condition to the case statement and yet another method to our `Player` class—in other words, it would require _modifying_ the source code. This growing case statement adds unecessary dependencies to `Player`, making it vulernable to unintended side effects. So how can we extend our code with the desired behaviour without making any modifications?

The OCP offers us a solution, and we're going to implement it with the strategy pattern.

{% highlight ruby %}
class Player
  attr_reader :move_getter

  def initialize(move_getter:)
    @move_getter = move_getter
  end

  def take_turn(board:)
    location = move_getter.get_move
    board.get_tile(location).destroy
  end
end

class MoveGetter
  def get_move
    # ask for a move via the user interface
  end
end

class RandomMoveGetter
  def get_move
    # randomly generate a move
  end
end
{% endhighlight %}

Now, when we add our “smart” computer player to the game, we simply create a new `SmartMoveGetter` class that subscribes to the same `.get_move` interface. This allows the `Player` class to remain untouched.

The authors of PPP offer a familiar caveat—do not apply the OCP until you have a reason to do so. In their words: _resisting premature abstraction is as important as abstraction itself_. If our battleship game only permitted human players, we would be foolish to implement the OCP in the hopes that one day we would need other types of players; however, once we had both human and computer players, we had a good reason to use the OCP via the strategy pattern.
