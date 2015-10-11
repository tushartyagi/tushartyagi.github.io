Title: Iteration in Python
Date: 2015-03-15
Category: Python
Tags: iteration, python
Slug: iteration-in-python
Authors: Tushar Tyagi
Summary: Iteration in Python


# Let's start Iterating in our own class

Almost all the built-in classes in Python allows us to iterate using `for..in`
syntax, and the language allows us to extend the user created classes to give
the same functionality using minimum coding. Let's see how we can achive the
same.

Any object which has to provide iteration is called an iterable, and to be one
it has to implement two methods:

* \_\_iter__()
* \_\_next__()

Whenever you use `for..in` loop, internally `__iter__` method is called, and it
has to return an iterable object. Since we are implementing this functionality
on some class, returning an object of this class, or `self` usually does the
trick.

The second method, `__next__` (or `next` in Python2) holds the logic which
returns the next object of the iterable. For instance, if we are iterating
over a list, [1,2,3,4], then calling `__next__` initially will return 1, then
the second call will return 2, and so on. Once the next-logic has exhausted
and there isn't any element to return, we have to raise a `StopIteration`
exception. for loop is designed to consume and digest this exception and
behave as if nothing happened, any other uncaught exception is passed through
it to the outer scope.

Let's create a class which takes in a list and returns a list with all the
elements doubled.

	:::python
	class DoubleTheList(object):
		def __init__(self, input_list):
			self.input_list = [2*x for x in input_list]
			self.input_list_length = len(self.input_list) - 1
			self.current_index = 0

		def __iter__(self):
			return self
		
		def __next__(self)
			if self.current_index <= self.input_list_length:
				current_val = self.input_list[self.current_index]
				self.current_index += 1
				return current_val
			else:
				raise StopIteration


Let's checkout our code:

	:::python
	> dbl_list = DoubleTheList([1,2,3,4]) 
	> for x in dbl_list: print(x)
	2
	4
	6
	8

	> for x in dbl_list: print(x)
	# Ouch! No output

## Don't depend on self

What happened??? In comparision to built-in iterators like lists, the behavior
of our object is totally differnt:

	:::python
	> x = [1,2,3]
	> for elem in x: print(elem)
	1
	2
	3

	> for elem in x: print(elem)
	1
	2
	3

If we iterate over a list (or tuple, or dict) we can keep on iterating again
and again.

As I said, `for` calls the `__iter__` method every time the loop is
initialised, and once it gets the object it calls the `__next__` method until
it encounters a `StopIteration` exception. So, then the fault lies in our
`__iter__()` method. If you see, although we are returning an object,  once
the `__next__` logic is exhausted in the object, it doesn't get renewed.

So instead of returning `self`, we have to return a new instance so that `for`
can start working with a fresh iteration. This requires a small change in our
code:

	:::python
	class DoubleTheList(object):
		def __init__(self, input_list):
			self.input_list = input_list
			self.input_list_length = len(self.input_list) - 1
			self.current_index = 0

		def __iter__(self):
			return DoubleTheList(self.input_list)
		
		def __next__(self)
			if self.current_index <= self.input_list_length:
				current_val = 2 * self.input_list[self.current_index]
				self.current_index += 1
				return current_val
			else:
				raise StopIteration


Let's test this thing:

	:::python
	> dbl_list = DoubleTheList([1,2,3,4]) 
	> for x in dbl_list: print(x)
	2
	4
	6
	8

	> for x in dbl_list: print(x)
	2
	4
	6
	8


### Parting Words ###
So it's clear that implementing iteration in Python is pretty simple and
straightforwards, like a lot of other things in the language. 
