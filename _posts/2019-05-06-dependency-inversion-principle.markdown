---
layout: post
title:  "SOLID: Dependency Inversion Principle"
date:   2019-05-06
---

We can place an application's modules along a spectrum of higher-level to lower-level behaviours—the higher-level ones are the business models of the program that contain important policy decisions, while the more detailed, lower-level ones contain implementation details. It's a difference between knowing _what_ should be done versus _how_ it should be done.

It may seem obvious that high-level modules should not be dependent upon low-level ones—we don't want the implementation details influencing important business decisions—however, it is a common result of traditional procedural design. The Dependency Inversion Principle (DIP) encourages us to _invert_ the direction of these dependencies by having high-level modules depend on abstractions instead of low-level modules.

It is not uncommon for a class to violate multiple SOLID principles at once, and classes that violate the Open-Closed Principle (OCP) are frequently offenders of DIP violations, too. In fact, we're going to borrow the code example from the prior [post on the OCP]({{ site.baseurl }}{% post_url 2019-04-22-open-closed-principle %}) to also demonstrate a violation of the DIP.

As a refresher, this code is from my Ruby implementation of Battleship. I expect both human and computer players to place a move on a board during their respective turns; however, I want the human players to supply the location of their next move via the user interface, and I want the computer players to randomly generate a location. Here’s an implementation this feature that violates the DIP.

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

We've already discussed how this code violates the OCP, but it also violates the rules of the DIP. In the context of the Battleship game application, `Player` is a fairly high-level class; yet here it is clearly dependent upon the implementation of getting a move for a human or a computer player. We can limit this dependency by creating an abstraction in the same way that we did when we addressed the OCP violation.

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

Now that the high-level `Player` depends on the abstract `get_move` duck type, it no longer knows anything about _how_ to get a move for a player.

The authors of _Agile Software Development: Principles, Patterns, and Practices_ (PPP) warn that despite our best efforts to have our classes depend on abstractions rather than concretions, a program is usually going to violate the DIP at least once. After all, _something_ has to instantiate the concrete classes.

The authors of PPP further explain that there is no real reason for adhering to the DIP for classes that are concrete but nonvolatile. If a concrete class is not going to change and we're not going to create any derivatives of it, then it does very little harm to depend on it. Depending on these concrete but stable classes will save us from introducing unnecessary complexity into our programs.
