---
layout: post
title:  "Design Techniques: Duck Typing 🦆"
date:   2019-03-21
categories: ruby
---

## What is a duck type?

Thinking about your application in terms of interfaces rather than classes is paramount to good object-oriented design. A public interface is comprised of the messages an object can receive from other objects, but interfaces can cut across classes, and an object can have many interfaces.

(describe duck typing)

## An example of duck typing

In my Ruby implementation of a Battleship game, I've been using duck typing to make my application more flexible.

When the Battleship program is run, the first thing it does is allow the users to place their ships on a board. For the human players, this involves asking them for the initial location of a ship, and then for the direction that the ship will point in (vertical or horizontal).

I handle the acquisition of these two pieces of data in separate classes: `GetShipLocation` and `GetShipDirection`:

{% highlight ruby %}
Class GetShipLocation
  def perform(ship)
    # ...
  end
end

Class GetShipDirection
  def perform(ship)
    # ...
  end
end
{% endhighlight %}

The shared `#perform` methods comprise a duck type, which allows us to use these objects without caring which class they belong to, only that they respond to the `.perform` message. While the `#perform` methods share the same name, they do not share the same behaviour.

I make use of this `#perform` interface in the `GetShipPlacementData` class:

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

get_ship_placement_data = GetShipPlacementData.new(services: [ GetShipLocation.new, GetShipDirection.new ])
get_ship_placement_data.perform(Ship.new) #=> { ... }
{% endhighlight %}

When the instance of `GetShipPlacementData` receives `.perform` with a `Ship` object, it calls `.perform` on each service. It expects each `.perform` call to return a hash (or rather, it expects it to return an object that responds to `.merge!`). This expectation placed on the return value of `.perform` is also a part of the interface's "contract".

## How to spot hidden duck types

Let's rewrite the code above to rely on the objects' classes rather than the duck type interface:

{% highlight ruby %}
Class GetShipLocation
  def get_location(ship)
    # ...
  end
end

Class GetShipDirection
  def get_direction(ship)
    # ...
  end
end
{% endhighlight %}

These classes no longer share an interface via the `#perform` message, so `GetShipPlacementData#perform` has to handle each object in its `services` array differently:

{% highlight ruby %}
class GetShipPlacementData
  attr_reader :services
  private :services

  def initialize(services:)
    @services = services
  end

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

This is the sort of pattern you should keep an eye out for——any time you 

## Why use duck typing?

what if I want to extend code above?

many more dependencies in code above - knows the names of two classes, the names of two different messages they receive as well as the arguments they accept. Having all this extra knowledge of other classes is risky——distant changes could cause unwanted side effects in the `GetShipPlacementData#perform` method.

Duck typing makes your code more tolerant of change.

If a ship’s placement data ever requires getting another piece of information, it would not be difficult to simply create and pass in another object that subscribes to this same interface.

For example, pretty soon I am going to have to make a `ComputerPlayer` that can place it’s own ships on the board; however, the process of getting each ship’s position is going to look pretty different to how it is done for a human player. Instead of accepting an array of services that print out instructions and wait for responses from a user, a new instance of `GetShipPlacementData` will be able to accept an array of services that randomly select locations and directions for its ships.

(makes the code more flexible - Pg 94 costs of concretion vs costs of abstraction)

