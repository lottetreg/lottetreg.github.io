---
layout: post
title:  "Design Techniques: Classical Inheritance"
date:   2019-03-28
categories: ruby
---

My previous [post about duck typing]({{ site.baseurl }}{% post_url 2019-03-21-design-techniques-duck-typing %}) showed duck-typed objects responding to the same message. But what if objects share not only the same message, but also the same behaviour? In order to keep our code DRY, we should extract this common behaviour to a single place in our code. One way to do this is with inheritance.

## What is inheritance?

In Ruby, classical inheritance and sharing behaviour via modules are both mechanisms for inheritance, which, at its core, is an implementation of _automatic message delegation_. Inheritance allows us to define a relationship between two objects, where if the first object receives a messge it doesn't understand, it will automatically forward (or delegate) that message to the second object. In this scenario the first object is the _subclass_ while the second is the _superclass_. In Ruby, while an object can have multiple subclasses, each object may only have one superclass. This is known as _single inheritance_.

An object can inherit from another object that inherits from another object, and so on, culminating in a chain of classes that can receive forwarded messages from any objects below them. Methods defined at the top of this look-up chain have widespread influence over the objects that sit below them—changes to these methods immediately ripple down the inheritance hierarchy, meaning you can accomplish big changes with small changes to the code; however, this also means that small changes can potentially break things. Using inheritance properly, therefore, is of the highest priority and requires adhering to specific coding techniques.

## Classical inheritance best practices

I have some code in my Ruby Battleship game that is responsible for returning an array of coordinates that represent a ship's placement on the board. It calculates these coordinates by taking the ship's starting position and adding the ship's length to it. If the ship is being placed vertically, it adds the length down the rows. And if it's being placed horizontally, it adds the length across the columns. I started out with two classes, one for vertical placements and another for horizontal ones.

{% highlight ruby %}
class VerticalCoordinatesBuilder
  attr_reader :first_row, :first_column, :length

  def initialize(first_row:, first_column:, length:)
    @first_row = first_row
    @first_column = first_column
    @length = length
  end

  def build
    array_of_rows.map do |row|
      { row: row, column: first_column }
    end
  end

  def array_of_rows
    Array (first_row..last_row)
  end

  def last_row
    (first_row + length) - 1
  end
end

class HorizontalCoordinatesBuilder
  attr_reader :first_row, :first_column, :length

  def initialize(first_row:, first_column:, length:)
    @first_row = first_row
    @first_column = first_column
    @length = length
  end

  def build
    array_of_columns.map do |column|
      { row: first_row, column: column }
    end
  end

  def array_of_columns
    Array (first_column..last_column)
  end

  def last_column
    (first_column + length) - 1
  end
end
{% endhighlight %}

These two classes obviously share an interface—they both respond to the `build` message and are even initialized in the same way. If we look closer, we see that the two `build` methods are also very similar. You get the sense that these two objects are strongly related with common behaviour, but they differ along a single dimension (vertical vs horizontal). This is the exact problem that inheritance solves—let's extract the common behaviour above to a single `CoordinatesBuilder` class.

{% highlight ruby %}
class CoordinatesBuilder
  attr_reader :first_row, :first_column, :length

  def initialize(first_row:, first_column:, length:)
    @first_row = first_row
    @first_column = first_column
    @length = length
  end

  def build
    array.map(&build_coordinate_pair)
  end

  def array
    raise "Implement `#array` in the #{self.class} class!"
  end

  def build_coordinate_pair
    raise "Implement `#build_coordinate_pair` in the #{self.class} class!"
  end

  def last(start)
    (start + length) - 1
  end
end

class HorizontalCoordinatesBuilder < CoordinatesBuilder
  def array
    Array (first_column..last(first_column))
  end

  def build_coordinate_pair
    Proc.new { |column| { row: first_row, column: column }}
  end
end

class VerticalCoordinatesBuilder < CoordinatesBuilder
  def array
    Array (first_row..last(first_row))
  end

  def build_coordinate_pair
    Proc.new { |row| { row: row, column: first_column }}
  end
end
{% endhighlight %}

Note: It is recommended that you wait until you have at least three potential subclasses before you try to extract a common superclass, but for the purpose of this post I've gone ahead with only the two.

### Separate abstractions and specializations

When using inheritance, ensure that your objects have a generalization-specialization relationship. The `CoordinatesBuilder` above provides the general behaviour for iterating over an array of rows and columns and returning an array of coordinate hashes. The subclasses that inherit from it are responsible for providing the specializations; namely, which array to loop over, and what function to apply to each element in the array.

One way to ensure your inherited code maintains this separation of abstractions and specializations is to use the _template method pattern_. We see this pattern in the code above—`CoordinatesBuilder` provides the general algorithm for building the array of coordinates, but it allows its subclasses to influence this algorithm by expecting them to implement the `array` and `build_coordinate_pair` methods. These are the _template methods_, and as a rule, the superclass should implement every template method it calls, even if it expects the subclasses to override them.

Defining every template method in the superclass may feel redundant, but it provides valuable documentation of the subclass-superclass relationship requirements. If the methods were missing from both the superclass and a subclass, the developer would only see a potentially confusing `NameError: undefined local variable or method...` error when a subclass object received those messages. Raising specific error messages in the superclass methods makes it clear to the developer that the subclasses are expected to implement these methods themselves.

### Decoupling subclasses from superclasses

Part of keeping your subclasses and superclasses loosely coupled is avoiding the use of `super`.

{% highlight ruby %}
class CoordinatesBuilder
  attr_reader :first_row, :first_column, :length

  def initialize(first_row:, first_column:, length:)
    @first_row = first_row
    @first_column = first_column
    @length = length
  end

  def build(array, block)
    array.map(&block)
  end

  # nothing else has changed
end

class HorizontalCoordinatesBuilder < CoordinatesBuilder
  def build
    super(array_of_columns, build_coordinate_pair)
  end

  def array_of_columns
    Array (first_column..last(first_column))
  end

  def build_coordinate_pair
    Proc.new { |column| { row: first_row, column: column }}
  end
end
{% endhighlight %}

`super` adds an additional, unneccessary dependency by forcing a subclass to know something about how its superclass works. All subclasses will now know how the algorithm in `CoordinatesBuilder` works. This use of `super` also leads to code duplication by requiring all current and future subclasses to call `super` in the same place and in the same way.

Instead of `super`, use _hook messages_ as we did in the previous example with our `array` and a `build_coordinate_pair` methods. These methods remove knowledge of the coordinate-building algorithm from the subclasses and hand control back to the parent `CoordinatesBuilder` class.

## Recognizing where to use inheritence

Your code might resemble the example at the beginning of this post, where the `HorizontalCoordinatesBuilder` and `VerticalCoordinatesBuilder` classes were essentially the same thing but they differed along a single dimension. This indicates that inheritance may be a good solution.

Another indication that you should consider inheritance is if you have a single class that is trying to account for multiple specializations.

{% highlight ruby %}
class CoordinatesBuilder
  attr_reader :first_row, :first_column, :length, :direction

  def initialize(first_row:, first_column:, length:, direction:)
    @first_row = first_row
    @first_column = first_column
    @length = length
    @direction = direction
  end

  def build
    case direction
    when 'vertical'
      build_vertical_coordinates
    when 'horizontal'
      build_horizontal_coordinates
    end
  end

  def build_vertical_coordinates
    array_of_rows.map { |row| { row: row, column: first_column }}
  end

  def build_horizontal_coordinates
    array_of_columns.map { |column| { row: first_row, column: column }}
  end

  def array_of_rows
    Array (first_row..last_row)
  end

  def array_of_columns
    Array (first_column..last_column)
  end
end
{% endhighlight %}

This case statement should remind you of the antipattern we saw in the [previous post on duck typing]({{ site.baseurl }}{% post_url 2019-03-21-design-techniques-duck-typing %}); however, instead of asking an object what class it belongs to, here we are checking the value of an attribute on `self` to determine which message to send to `self`. Checking for the value of `direction` inside the `build` method implies that `CoordinatesBuilder` has too many responsibilities—it's essentially made up of two objects that should be broken out into their own subtypes. With this case statement, the `CoordinatesBuilder` class must be changed every time you add a new direction (maybe we'll want to place ships diagonally in the future), instead of simply creating a new subclass that would extend the inheritance hierarchy.

If you think that inheritance might the right solution for your problem, think about the relationships you'd be creating and ensure that every subtype _is a_ type of its parent (e.g. a `HorizontalCoordinatesBuilder` _is a_ `CoordinatesBuilder`). If the relationship is better described as _behaves-like-a_, you should instead look into behaviour sharing via modules; if it is a _has-a_ relationship, you would be better off using composition.
