---
layout: post
title:  "Why Algorithms Matter"
date:   2020-08-07
---

The second chapter of [A Common-Sense Guide to Data Structures and Algorithms](https://pragprog.com/titles/jwdsal2/) shows us why we should consider which algorithms are best for our programs.

In computing, an algorithm is a particular process for going about an operation. (We covered the four primary operations—reading, searching, inserting, and deleting—in [the last post]({{ site.baseurl }}{% post_url 2020-08-01-why-data-structures-matter %}).) There are many algorithms to choose from to achieve a specific operation.

# Ordered Arrays

Ordered arrays, it should come as no great surprise, have their elements in order. Unlike a regular array, an ordered array has to determine _where_ to place a new value before inserting it. Inserting a value into an ordered array, therefore, first requires a search to find this position.

# Linear Search on Ordered Arrays

Up until now, we've only used linear searches on regular arrays, where we look inside every single element in the array for the matching value. We only stop searching when we either find the value or we reach the end of the array.

When we perform a linear search on an ordered array, however, we can stop as soon as we find the value, _or_ when we find a value that is greater than the one we're looking for.

Here is a Ruby implementation of a linear search on an ordered array:

{% highlight ruby %}
def linear_search(array, value)
  array.each_with_index do |element, index|
    if element == value
      return index
    elsif element > value
      break
    end
  end

  return nil
end
{% endhighlight %}

We start by moving through each array element. If we come across an element that matches our search value, we return that element's index. If we come across an element that is greater than our search value, we break the loop and return `nil`.

In most situations, it appears that performing a linear search on an ordered array will take fewer steps than performing it on a regular array. However, if the element we're searching for is at the very end of an ordered array, we will still have to loop through every cell.

At first glance, therefore, ordered arrays are not that much more efficient than regular arrays—but this is because we are still using a linear search. Ordered arrays are powerful because they allow us to use an alternative searching algorithm called a binary search, which is _much_ faster than a linear one.

# Binary Search

With a binary search, we repeatedly eliminate half of the array until we either find the value, or we determine that it is not in the array. This search can only be performed on ordered arrays, not standard ones.

Here is an implementation of binary search in Ruby:

{% highlight ruby %}
def binary_search(array, value)
  lower_bound = 0
  upper_bound = array.length - 1

  while lower_bound <= upper_bound do
    midpoint = (lower_bound + upper_bound) / 2
    value_at_midpoint = array[midpoint]

    if value < value_at_midpoint
      upper_bound = midpoint - 1
    elsif value > value_at_midpoint
      lower_bound = midpoint + 1
    elsif value == value_at_midpoint
      return midpoint
    end
  end

  return nil
end
{% endhighlight %}

We start off setting the lower bound to the first element and the upper bound to the last one. We then begin a loop in which we keep inspecting the middle value between the lower and upper bounds.

If the search value is lower than the midpoint value, we set the upper bound to be one place below that midpoint. If the search value is greater than the midpoint value, we set the lower bound to be one place above the midpoint. And if the values match, we simply return the midpoint.

If the lower and upper bounds cross each other, then we know that the search value is not in the array, and we return `nil`.

# Binary Search vs. Linear Search

With a linear search, the maximum number of steps is equal to the number of elements. This means it would take a maximum of 100 steps to search an array of 100 elements.

For that same array of 100 elements, it would take a maximum of 7 steps to perform a binary search. Every time you _double_ the number of elements in an array, the binary search algorithm only adds a maximum of one more step.

The difference in efficiency between the binary search and the linear search is incredible; however, a binary search only really gives us an advantage when it comes to large arrays. For example, performing a binary search on an array of 1 million elements would only take _20 steps_.

Ordered arrays are not faster in every respect—we've already seen that inserting a value into an ordered array is slower, since the insertion needs to first perform a search. But while you may have a slightly slower insertion, you will have a much faster search. It is a trade-off, and you must analyze your application to see which array will serve your code the best. For example, if you anticipate frequently adding data, but rarely searching it, a standard array may be a better choice.
