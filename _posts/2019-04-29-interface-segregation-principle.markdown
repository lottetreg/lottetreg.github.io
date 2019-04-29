---
layout: post
title:  "SOLID: Interface Segregation Principle"
date:   2019-04-29
---

"Fat" interfaces have methods that are used by some, but not all of their clients. Discussions around the Interface Segregation Principle (ISP) tend to use the concept of "interfaces" that we see in Java and C++ (the abstract types used to specify a behaviour that classes must implement). While we may not have these same interfaces in Ruby, we've seen how important the concept of interfaces is in object-oriented design, and therefore the ISP is an important concept to understand in Ruby programming.

In my Ruby Battleship game, I have two classes that present a board by returning a "stringified" version of it—one of these simply returns the stringified board as is, while the other first "redacts" sensitive information, such as the locations of the other player's ships. Both of these board presenter classes respond to `.present`, and the sharing of this message is an interface that clients rely on.

{% highlight ruby %}
module BoardPresenterHelpers
  def stringify(board)
    # ...
  end
end

class BoardPresenter
  include BoardPresenterHelpers

  def present(board:)
    stringify(board)
  end
end


class RedactedBoardPresenter
  include BoardPresenterHelpers

  def present(board:)
    redacted_board = redact_board(board)
    stringify(redacted_board)
  end

  def redact_board(board)
    # ...
  end
end
{% endhighlight %}

This interface is used in the following way.

{% highlight ruby %}
board_presenters.each do |board_presenter|
  print(board_presenter.present(board: board))
end
{% endhighlight %}

This array of `board_presenters` could contain instances of both board presenter classes. Now let's look at an implementation of this interface that violates the ISP.

{% highlight ruby %}
module BoardPresenterHelpers
  def present(board:, redacted: false)
    board = redacted ? redact_board(board) : board
    stringify(board)
  end

  def redact_board(board)
    # ...
  end

  def stringify(board)
    # ...
  end
end

class BoardPresenter
  include BoardPresenterHelpers
end

class RedactedBoardPresenter
  include BoardPresenterHelpers
end
{% endhighlight %}

The authors of _Agile Software Development: Principles, Patterns, and Practices_ (PPP) say that clients should not be forced to depend on methods that they do not use. We see above that the `BoardPresenter` class is dependent upon the `#redact_board` method, even though it will never actually use it. The `RedactedBoardPresenter` does use the `#redact_board` method, however, and therefore it might force changes upon the method. These changes could affect the `BoardPresenter` class, meaning that these two classes are coupled in an unneccessary way.

We can avoid violating the ISP by breaking up these "fat" interfaces into mulitple client-specific interfaces, as we had done in our original code sample where each board presenter class implemented its own `#present` method. Defining separate `#present` methods on the two board presenter classes prevented them from being coupled together through a shared knowledge of the `.redact_board` method.
