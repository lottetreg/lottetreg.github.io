---
layout: post
title:  "The Four Rules of Simple Design"
date:   2019-05-13
---

Kent Beck developed the four rules of simple design in the late 1990's. The rules, in order of highest priority to lowest, are the following:

1) Passes the tests<br>
2) Expresses intent<br>
3) Has no duplication<br>
4) Has the fewest elements

## Passes the Tests
This first rule is the most important—first and foremost your code should work as expected. If you follow with this guideline, you will strive to have all your tests running green by the time you're ready to merge your work. In order to do their job properly, your tests need to comprehensively and accurately represent the behaviour of your program.

## Expresses Intent
This second-most important guideline reminds us that our code is going to be read many more times than it's going to be edited (both by our future selves and by other developers). With this in mind, our code needs to be easy to understand. We can achieve this by using a number of best practices, including using clear and consistent names in our programs, having our methods and classes maintain single responsibilities, and avoiding deeply nested inheritance hierarchies, to name a few.

## Has No Duplication
"Don't Repeat Yourself" (DRY) may sometimes feel at odds with the previous suggestion to express intent through your code. For example, when we believe that our tests should serve as documentation for our production code, we may be tempted to eschew the DRY principle and repeat commonalities among tests. This can cause problems if we make changes that require us to make the same edits in multiple places. In his [post on the four rules of simple design](https://martinfowler.com/bliki/BeckDesignRules.html), Martin Fowler says "...adding duplication to increase clarity is often papering over a problem, when it would be better to solve it." In our tests, therefore, it may be better to consolidate the shared code into well-named methods, limiting duplication without risking the readability of our tests. Martin Fowler suggests that this rule has the same priority level as the previous one (express intent), as they naturally feed off each other when refining code.

## Has the Fewest Elements
According to this final rule, any code that does not satisfy the following rules should be removed. Notice that none of these four rules are concerned with keeping our code flexible (through the use of abstractions, for example), which we may have expected to see if we've spent time researching object-oriented design patterns and the SOLID principles. This final rule reminds us that an increase in flexibility leads to an increase in complexity, and suggests that it can actually make systems harder to modify and therefore less flexible in practice.
