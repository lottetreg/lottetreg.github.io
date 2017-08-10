---
layout: post
title:  "Under the Hood of Ruby's Shuffle"
date:   2017-08-09
categories: ruby
---

I've been working on an cards API that so far has the ability to create, show, and shuffle a deck of cards. Ruby provides a nifty method, `shuffle`, that handles the task of randomly ordering elements in an array, and I was curious about how it does this.

Ruby source code is of course written in c, and the c method that's utilized when you call `.shuffle` is `rb_ary_shuffle_bang`.

{% highlight c linenos %}
static VALUE
rb_ary_shuffle_bang(int argc, VALUE *argv, VALUE ary) 
{
  VALUE opts, randgen = rb_cRandom;
  long i, len;

  if (OPTHASH_GIVEN_P(opts)) {
    VALUE rnd;
    ID keyword_ids[1];

    keyword_ids[0] = id_random;
    rb_get_kwargs(opts, keyword_ids, 0, 1, &rnd);
    if (rnd != Qundef) {
      randgen = rnd;
    }
  }

  rb_check_arity(argc, 0, 0);
  rb_ary_modify(ary);

  i = len = RARRAY_LEN(ary);
  RARRAY_PTR_USE(ary, ptr, {
    while (i) {
      long j = RAND_UPTO(i);
      VALUE tmp;
      if (len != RARRAY_LEN(ary) || ptr != RARRAY_CONST_PTR(ary)) {
        rb_raise(rb_eRuntimeError, "modified during shuffle");
      }
      tmp = ptr[--i];
      ptr[i] = ptr[j];
      ptr[j] = tmp;
    }
  }); /* WB: no new reference */
  return ary;
}
{% endhighlight %}

To figure out how it works, you only really need to look at lines 21-35. 

We have a while loop that uses the length of the array (`i`). Our loop moves backwards through the array, starting with the last element.

On line 24 it randomly selects a number between 0 and the length of the array, and stores this number in the variable `j`. (If our array was `[1, 2, 3]` with a length of 3, it would randomly select a number bewteen 0 and 2.)

On line 29, `ptr[--i]` returns the value of the element our while loop is currently on. We store this value in a variable, `tmp`. Because this line decerements the value of `i`, the while loop will continue to move down through the array. 

On line 30 we set the value of our current element to that of the randomly selected element (`ptr[j]`). On the next line we finally set the value of the randomly selected element to what was originally in our current element, but is now stored in `tmp`.

To summarize, the `rb_ary_shuffle_bang` method loops through the array and switches the value of each element with that of another randomly selected element.