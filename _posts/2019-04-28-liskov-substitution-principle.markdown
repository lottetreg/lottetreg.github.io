---
layout: post
title:  "SOLID: Liskov Substitution Principle"
date:   2019-04-28
categories: ruby
---

We saw in the previous [post on the Open-Closed Principle]({{ site.baseurl }}{% post_url 2019-04-22-open-closed-principle %}) that polymorphism is one of the most powerful tools in our object-oriented design tool belt. Having classes inherit from each other is one way to achieve polymorphism, and the The Liskov Substitution Principle (LSP) provides important guidelines for building inheritance hierarchies without making our code brittle.

The LSP requires that a parent class can be replaced by any of its subclasses without incurring unexpected behaviour. Let's look at an example of some code that violates the LSP.

{% highlight ruby %}
class Rectangle
  attr_accessor :height, :width
end

class Square < Rectangle
  attr_reader :height, :width

  def height=(height)
    super(height)
    self.width = height
  end

  def width=(width)
    super(width)
    self.height = width
  end
end
{% endhighlight %}

You've heard it many times—a rectangle isn't a square, but a square is always a rectangle. With that in mind, this code might seem reasonable at first glance. We know that a square's width will always be the same as its height, so overriding the `height` and `width` setter methods to also assign the other attributes makes sense. We could say that this design is _self consistent_; however, the authors of _Agile Software Development: Principles, Patterns, and Practices_ warn that a self-consistent design is not necessarily consistent with all of its users.

To demonstrate this, we'll add another method to our `Rectangle` class.

{% highlight ruby %}
class Rectangle
  attr_accessor :height, :width

  def area
    width * height
  end
end

class Square < Rectangle
  # ...
end
{% endhighlight %}

And now let's write a parameterized RSpec test for the new `#area` method.

{% highlight ruby %}
shared_examples "a Rectangle" do |rectangle|
  it "has an area that is the product of width and height" do
    rectangle.height = 10
    rectangle.width = 5

    expect(rectangle.area).to eq(50)
  end
end

it_behaves_like 'a Rectangle', Rectangle.new # Success
it_behaves_like 'a Rectangle', Square.new # Failure
# it expected `rectangle.area` to equal 50, but got 25 since line 4 resets the height to 5
{% endhighlight %}

It should have been safe to assume that changing the width of a rectangle would not affect its height, and the writer of this test was completely reasonable to expect an instance of `Square` to behave like a `Rectangle`. Since `Square` inherits from `Rectangle`, a `Square` _is_ a `Rectangle`.

This is a clear violation of the LSP, and there are other smells to look out for, too. For example, if a subclass overrides a method on its super with an empty (a.k.a. degenerative) method, the subclass is not _really_ of the same type as its parent. Similarly, if a subclass method raises an exception that its super method does not, then that subclass is not substitutable for its parent.

It's important to note that a violation of LSP is not necessarily poor design. For example, we saw in my previous [post on Classical Inheritance]({{ site.baseurl }}{% post_url 2019-03-28-design-techniques-inheritance-with-the-template-method-pattern %}) that a superclass may raise an error prompting the developer to override a method in a subclass. Here's a short example of the same idea.

{% highlight ruby %}
class Rectangle
  def do_something
    raise "Implement `#do_something` in the #{self.class} class!"
  end
end

class Square < Rectangle
  def do_something
    # does something
  end
end
{% endhighlight %}

Technically, this is a violation of the LSP—by overriding the `#do_something` method, `Square` is no longer strictly substitutable for `Rectangle`; however, we can reasonably assume that whichever client uses `Square` is not going to _expect_ it to raise that error; rather, it's going to expect it to behave in whatever way it has been overridden. You must consider the way in which a client _expects_ a model to work and the reasonable assumptions other users will make about it when determining the validity of the design.

Of course, it is very difficult to predict what reasonable assumptions other users will make about your models, and trying to anticipate them prematurely will likely lead to unnecessary complexity in your code. As we've heard in all of our previous explorations into the SOLID principles, it is best to defer all but the most obvious LSP violations until you have good reason to address them.
