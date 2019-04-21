---
layout: post
title:  "SOLID: Single-Responsibility Principle"
date:   2019-04-21
categories: ruby
---

Without even knowing it’s definition, the Single-Responsibility Principle (SRP) sounds like it would be a good thing for your code; after all, anything that has one responsibility is likely to be smaller, less complex, and more maintainable than something that has multiple responsibilities. But SRP gives us even more than that.

The authors of _Agile Software Development: Principles, Patterns, and Practices_ (PPP) explain that responsibilities that live within the same class are coupled, and this coupling can impair our ability to make otherwise reasonable changes. In classes with coupled responsibilities, changes to one responsibility may adversely affect the class’s ability to uphold another.

Understanding what we mean by “responsibility”, however, can be difficult. According to the authors of PPP, responsibility is “a reason for change”—if you can think of multiple motives for changing a class, then that class has multiple responsibilities.

_An important note: when we use the term "changes", we mean changes to the actual code of the class, not changes to the state of the object at run time._

Let’s look at some code that violates SRP. I have a class in my Ruby implementation of Battleship that controls the flow of the game—it determines whose turn it is, tells the players to take their turns, and ends the game when a player has lost all of their ships.

{% highlight ruby %}
class Game
  attr_reader :players
  attr_writer :turn_count

  def initialize(first_player:, second_player:)
    @players = [first_player, second_player]
  end

  def play
    until game_over?
      current_player.take_turn(board: current_opponent.board)
      switch_turn
    end
  end

  def game_over?
    players.any?(&:has_lost_all_ships?)
  end

  def current_player
    players[turn_count % players.size]
  end

  def current_opponent
    players.reverse[turn_count % players.size]
  end

  def switch_turn
    self.turn_count += 1
  end

  def turn_count
    @turn_count ||= 0
  end
end
{% endhighlight %}

This code violates SRP because of the multiple ways in which it might change. For example, if we wanted to end a game when a player "surrendered", we would have to edit the `game_over?` method to account for this case. And if we wanted to change something about the mechanics of a game's turns (maybe we want to allow more than two players), then the logic to determine who is the `current_player` and who is the `current_opponent` would have to be adjusted. Having two distinct ways in which this class could potentially change is a signal that we’re violating SRP.

To fix this, we can remove one of the responsibilities from the `Game` class and give it a new home.

{% highlight ruby %}
class Game
  attr_reader :players
  attr_writer :turn_count

  def initialize(first_player:, second_player:)
    @players = [first_player, second_player]
  end

  def play
    until GameRules.new.game_over?(self)
      current_player.take_turn(board: current_opponent.board)
      switch_turn
    end
  end

  def current_player
    players[turn_count % players.size]
  end

  def current_opponent
    players.reverse[turn_count % players.size]
  end

  def switch_turn
    self.turn_count += 1
  end

  def turn_count
    @turn_count ||= 0
  end
end

class GameRules
  def game_over?(game)
    game.players.any?(&:has_lost_all_ships?)
  end
end
{% endhighlight %}

Our `Game` class is now smaller and has one less responsibility, and the `GameRules` class has the single responsibility of determining whether a game is over. This code would make it easier for us to implement either of the two potential changes we came up with earlier.

We’ve just seen an example of how to address a violation of SRP, but that does not mean that we _should_ have done this refactoring. The authors of PPP warn that if an application is not _actually_ changing in a way that would cause the responsibilities to change at different times, a separation of responsibilities adds needless complexity. Until we have a real reason to, there is no need to separate them.
