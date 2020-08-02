---
layout: post
title:  "Why Data Structures Matter"
date:   2020-08-01
---

The first chapter of Jay Wengrow's [A Common-Sense Guide to Data Structures and Algorithms](https://pragprog.com/titles/jwdsal2/) explains how to think about the time complexity of data structures.

The purpose of computer programs is to receive, manipulate, and return information. This information, all the way down to the most basic numbers and strings, is called data, and we organize this data inside data structures. This organization can significantly impact how fast your code runs, or if it even runs at all.

## Operations

There are four basic ways, or operations, in which most data structures are used, and we can analyze these operations to measure a data structure's performance.

The four operations are:

_Read_: Reading refers to looking up an element from a known spot inside the data structure.

_Search_: Searching refers to moving through the data structure to find a particular value.

_Insert_: Inserting refers to adding another value to the data structure.

_Delete_: Deleting refers to removing a value from the data structure.

When assessing a data structure's performance, we can analyze how fast each of these operations are when applied to that data structure. When measuring this speed, however, we do not refer to pure time, since a number of factors could affect how long an operation actually takes. Instead, we refer to the number of _steps_ an operation takes. Measuring the speed of an operation is also known as measuring its time complexity.

## Arrays

An array, which is simply a list of data elements, is one of the most basic data structures in computer science. Think of a computer's memory as a grid of cells—when a program declares an array, it allocates a contiguous set of empty cells in this grid. Each cell has a memory address, represented by a number, and each cell's address is one greater than the cell's before it. The computer always knows the address of the first element in the array, as well as the total number of elements.

Let's see how an array performs with the operations we described above.

# Reading from an array

Reading from an array only takes one step. This is because the computer can immediately jump to a given index and retrieve whatever value is there. If we request the element with an index of 2, the computer will take the address of the first array element, add 2 to it to calculate the address of the element we want, and go straight to that address to return the value.

One of the reasons that arrays are such powerful data structures is because they can look up the value of any index with such speed.

# Searching an array

Although there are different search algorithms (which we'll explore in later chapters), in this example we're going to use linear search. In a linear search, the computer moves through each cell until it finds the value it's looking for or reaches the end. If it does find the given value, it returns that element's index.

If the element we're searching for is located near the beginning of the array, this operation won't take very long. But when we're analyzing the performance of data structures and algorithms, we consider the worst-case scenarios. The worst-case scenario when searching an array is for the element we're looking for to either be last, or not there at all.

In either of these worst-case scenarios, the computer would have to look inside every element in the array; therefore, if our array has _N_ elements in it, a linear search would take _N_ steps. Reading from an array, which always takes one step, is clearly faster than searching an array.


# Inserting into an array

Insertion can be done in one step if the element is being added to the very end of the array. In this case, since the computer knows both the memory address of the first element and the length of the array, it can easily calculate the address of the cell it needs to add the element to. The insertion can then be completed in one step.

Inserting an element anywhere else in the array, though, is a different story—before the new element can be inserted, all of the elements ahead of it need to be shifted down. Each cell that needs to be shifted adds another step to the insertion process, so the worst-case scenario is inserting the element to the beginning of the array. If there are _N_ elements in the array, the insertion will take _N_ steps, plus the step of actually placing the element in the array. Inserting into an array, therefore, takes _N + 1_ steps.


# Deleting from an array

Deletion behaves like insertion, but in reverse. Arrays cannot have gaps, so if an element is deleted from anywhere except the very end of the array, all of the elements in subsequent cells need to be shifted up.

Like insertion, the worst-case scenario for deletion is removing the very first element. In this case, if there are _N_ array elements, and one is deleted, _N - 1_ elements need to be shifted up. We also have to include the step of actually deleting the element, so the total number of steps is _N - 1 + 1_, or simply _N_.


## Sets

Although there are different kinds of sets, in this chapter the author is only concerned with the array-based set. The only difference between this set and an array is that it prohibits duplicate values from being inserted into it. This data structure is useful if you need to ensure that your data values are unique, but it also has a different efficiency for one of the four primary operations.

While reading, searching, and deleting are the same for both arrays and sets, this is not the case for insertion. Because the set needs to confirm that it does not already contain the value before inserting it, it first runs a search. We've already seen that a search takes _N_ steps for an array with _N_ elements, and that a pure insertion takes _N + 1_ steps. Adding these together, we can see that a set insertion requires _2N + 1_ steps.

Sets are important if you need to ensure you have no duplicate data, but if you don't, an array may be more performant for insertions.

## Up next

In the next chapter we'll use what we've learned about time complexity to compare the performance of algorithms (even within the same data structures).
