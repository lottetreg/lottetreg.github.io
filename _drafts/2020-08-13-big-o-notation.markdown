---
layout: post
title:  "Big O Notation"
date:   2020-08-13
---

The last chapter of [A Common-Sense Guide to Data Structures and Algorithms](https://pragprog.com/titles/jwdsal2/) used the number of steps an algorithm takes to describe its performance, and the third chapter gives...

Big O answers the following question: how does the number of steps change as the data increases?

_O(1)_:

The number of steps stays constant as the amount of data scales. O(1) is also referred to as constant time

Even if it takes 3 steps,  as long as it will _always_ take three steps, the Big O Notation is still O(1). 

Reading from an array, insertion and deletion of an element at the end of an array


_O(N)_:

Also called linear time

Number of steps increases at the same rate as the number of array elements

Linear search, worst case scenario, when element is at the end of array

We already know that reading from an array takes one step, no matter how much data the array has. In Big O Notation, this is expressed as _O(1)_ (pronounced, "Oh of one"). Other operations that fall under the category of _O(1)_ are the insertion and deletion of an element at the end of an array.

The number of steps it takes to complete a linear search on an array, we've also seen, scales with the number of array elements. The appropriate 


# Constant Time vs Linear Time

An algorithm that takes 100 steps still has constant time if it always takes 100 steps, no matter the size of the data. While a one-step algorithm is more efficient than one with 100 steps, a 100-step algorithm is still more efficient than any linear algorithm.

Once the data array reaches 100 elements, both a constant 100-step algorithm and a linear algorithm will take the same number of steps (100). There will always be some number of elements in which the tides turn

(Image of graph) pg 64
