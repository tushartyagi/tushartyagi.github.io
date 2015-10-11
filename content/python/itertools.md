Title: Python Weekly Module: Itertools 
Date: 2015-04-10
Category: Python
Tags: itertools, python, weekly-module
Slug: python-wm-itertools
Authors: Tushar Tyagi
Summary: Python Itertools
Status: draft

# Python Weekly Module: Itertools

I started going through python modules so that I have a better grasp of the
standard library. Python comes with "batteries included", so why shouldn't I
use them?

I hope that one week will be sufficient for one module, it may be less in
some cases, then I'll try to figure out the best way to complete it in time,
or let it take it some time over the next week.

This is the first iteration of WeeklyModules, an idea taken from
[pymotw.com](http://pymotw.com/2/contents.html).

I'll also try to start with Rust Weekly Crate and Javascript Weekly Pattern
(since it only has functions for almost everything). I hope the by doing this,
I'll have some  better idea about these languages, and I do plan to create
something out of every module which makes sense in real world. I just hope I
don't run out of ideas, and/or steam.

Coming back, this week (Apr 10 2015) I'm gonna start with `itertools` which
kind of extends on my previous article on iterators.

### chain() ###
This method takes in iterators, iterates through each one, and returns the
elements. The iteration through iterators is serial, i.e. once the first
iterator is exhausted, it moves to the second one, and then through the
others to the last one.

We have three iterators here, and I'll use `chain` to go from one to another

	iter1 = 'abcd'
	iter2 = (x for x in range(5))
	iter3 = [x for x in 'word']

	for i in chain(iter1, iter2, iter3): print(i)
	a
	b
	c
	d
	0
	1
	2
	3
	4
	w
	o
	r
	d

### compress() ###
`compress` method takes in an two iterators, and maps the values of both
one-to-one and returns the value of first iterator if the value of second
iterator is `True`, the values corresponding to `False` are not returned.
Keep in mind that `False` can also be something like an empty dict, an
empty list, etc.

Compression halts if any iterator gets exhausted.

	isvowel = lambda x : x in ['a', 'e', 'i', 'o', 'u']


### *while() ###
This contains two methods, `dropwhile()` and `takewhile()`. Each method
expects two arguments:  a predicate function, and an  iterator.

`dropwhile` keeps on "dropping" or rejecting values until the predicate
function returns true, and then returns all the remaining values of the
iterator.

	[x for x in dropwhile(lambda x: x < 5

`takewhile` keeps on "taking" or listing the values until the predicate
function returns true, and then it terminates.

