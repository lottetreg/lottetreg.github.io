---
layout: post
title:  "Design Techniques: Duck Typing 🦆"
date:   2019-03-21
categories: ruby
---

You may already be familiar with the idea of public and private interfaces—they are the methods that are implemented within a class, and only the public ones are made available to other objects. But interfaces are not limited to single classes—some interfaces cut across multiple classes, and an object can have many interfaces.

We call these across-class interfaces duck types, and every duck-typed class responds to a set of the same messages. It is this set of messages themselves that define the duck type. Sandi Metz describes them further on page 61 of Practical Object-Oriented Design in Ruby:

> It's almost as if the interface defines a _vitual_ class; that is, any class that implements the required methods can act like the _interface_ kind of a thing.

Thinking about your application in terms of interfaces, including the more abstract duck types, is paramount to good object-oriented design.

## An example of duck typing

In my Ruby implementation of a Battleship game, I've been using duck typing to make my application more flexible.

When the Battleship program is run, the first thing it does is allow the users to place their ships on a board. For the human players, this involves asking them for the initial location of a ship, and then for the direction of the ship (vertical or horizontal).

I handle the acquisition of these two pieces of data in separate classes: `GetShipLocation` and `GetShipDirection`:

{% highlight ruby %}
class GetShipLocation
  def perform(ship)
    # ask the user for a location on the board
  end
end

class GetShipDirection
  def perform(ship)
    # ask the user for a direction
  end
end
{% endhighlight %}

The shared `perform` methods comprise a duck type, which allows us to use these objects without caring which class they belong to, only that they respond to the `perform` message. While the `perform` methods share the same name, they do not share the same behaviour.

I make use of this `perform` interface in the `GetShipPlacementData` class:

{% highlight ruby %}
class GetShipPlacementData
  attr_reader :services
  private :services

  def initialize(services:)
    @services = services
  end

  def perform(ship)
    services.each_with_object({}) do |service, results|
      result = service.perform(ship)
      results.merge!(result)
    end
  end
end

services = [ GetShipLocation.new, GetShipDirection.new ]
get_ship_placement_data = GetShipPlacementData.new(services: services)
get_ship_placement_data.perform(Ship.new) #=> { ... }
{% endhighlight %}

When the instance of `GetShipPlacementData` receives `perform` with a `Ship` object, it calls `perform` on each service. It expects each `perform` call to return a hash (or rather, it expects it to return an object that responds to `merge!`). This expectation placed on the return value of `perform` is also a part of the interface's contract.

## How to spot hidden duck types

Let's rewrite the code above to rely on the objects' classes rather than the duck type interface:

{% highlight ruby %}
class GetShipLocation
  def get_location(ship)
    # ...
  end
end

class GetShipDirection
  def get_direction(ship)
    # ...
  end
end
{% endhighlight %}

These classes no longer share an interface via the `perform` message, so `GetShipPlacementData#perform` has to handle each object in its `services` array differently:

{% highlight ruby %}
class GetShipPlacementData
  # ...

  def perform(ship)
    services.each_with_object({}) do |service, results|
      result = case service
        when GetShipLocation
          service.get_location(ship)
        when GetShipDirection
          service.get_direction(ship)
        end
      results.merge!(result)
    end
  end
end
{% endhighlight %}

This is just one of many common code patterns that indicate a duck type would be a better solution. In fact, any time you find yourself asking an object what it is, you should consider your reasoning.

Other ways of asking an object about it's class include sending it the `kind_of` and `is_a?` messages. `responds_to?` is another method to watch out for:

{% highlight ruby %}
class GetShipPlacementData
  # ...

  def perform(ship)
    services.each_with_object({}) do |service, results|
      result =
        if service.responds_to?(:get_location)
          service.get_location(ship)
        elsif service.responds_to?(:get_direction)
          service.get_direction(ship)
        end
      results.merge!(result)
    end
  end
end
{% endhighlight %}

In the example above, we may no longer be explicitly checking which class an object belongs to, but we are still very much expecting each object to be of a specific class.

## Why use duck typing?

The code examples above that do not make use of duck types are brittle and prone to breaking as a result of changes made to distant parts of the code base.

This is because checking for an object's class can introduce unnecessary dependencies. In the case statement example above, `GetShipPlacementData` knows not only the name of two other classes, but also the names of two methods on those classes and what kind of arguments they expect. If any of these qualities change, `GetShipPlacementData` would also have to be changed, or the `perform` method would not work as expected. And even though the `responds_to?` example is slightly better for not knowing about the `GetShipLocation` and `GetShipDirection` class names, it still knows too much about the individual method calls.

Furthermore, consider what it would take to extend these dependency-laden examples. If we wanted `GetShipPlacementData` to perform another service, it would require adding yet another check to the case statement, or another `if` statement to check whether the object responds to a particular message. Each addition would only further increase the number of `GetShipPlacementData`'s dependencies.

The code that does make use of duck typing, however, is much more tolerant of change. Not only does it have fewer dependencies, but it is easily extendable—if we ever need to get another piece of information to place a ship, it would not be difficult to simply create and pass in another object that subscribes to this `perform` interface.

For example, soon I am going to have a computer player that can place it’s own ships on the board; however, the process of getting each ship’s position is going to look very different to how it is done for a human player. Instead of printing out instructions and waiting for the responses from a user, a computer player will randomly select positions for its ships. To acheive this, a new instance of `GetShipPlacementData` will simply accept an array of other service objects that still respond to `perform`, but that return hashes of randomly selected position data:

{% highlight ruby %}
class GenerateShipLocation
  def perform(ship)
    # randomly select a location on the board
  end
end

class GenerateShipDirection
  def perform(ship)
    # randomly select a direction
  end
end

class GetShipPlacementData
  # nothing has changed here
end

services = [ GenerateShipLocation.new, GenerateShipDirection.new ]
get_ship_placement_data = GetShipPlacementData.new(services: services)
get_ship_placement_data.perform(Ship.new) #=> { ... }
{% endhighlight %}

If it feels like you are placing a lot of trust in the duck types to respond to a shared message in a particular way, you are right. But this is desired, and you should embrace that feeling. Trusting an interface shifts the sentiment of your code from knowing exactly how an object works and telling it how to behave, to telling it _I don't know who you are, but I trust you to do your job_. It is your responsibility as a programmer to make sure these objects are trustworthy.
