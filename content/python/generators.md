Title: Generators in Python
Date: 2015-01-15
Category: Python
Tags: generators, python, deep-dive
Slug: generators-in-python
Authors: Tushar Tyagi
Summary: Generators in Python

Python generators are functions that allows lazy evaluation of data, .i.e. data
is evaluated on-need basis. Unlike functions, which return the value and then
stop executing, generators simply _pause_ their execution and during the next
call continue the execution where they left, return another value and pause
again.

The simplest way of creating generators is using generator comprehensions,
which follow the format of list comprehensions but are surrounded with round
brackets:

	:::python
    # list comprehension:
	>>> lc = [x for x in range(2)]
	>>> lc⏎
	[0, 1]

    # generator comprehension
	>>> gc = (x for x in range(2)]
	>>> gc⏎
	<generator object <genexpr> at 0x23f8123>

A generator object is an iterable, i.e. it provides a method `__next__()` which
calls the next object of the iteration, or raises `StopIteration` exception if
there isn't any value left. Taking the example of `gc` object that we created
above:

	:::python
	>>> gc.__next__()⏎
	0
	>>> gc.__next__()⏎
	1
	>>> gc.__next__()⏎
	StopIteration...

Since it's an iterable, it can be used directly in the loop constructs. To show
this example, we'll have to create a new generator object. This is because once
the generator object raises `StopIteration`, it can't be used again, we have to
create a new object.

	:::python
	>>> gc1 = (x for x in range(2))
	>>> for value in gc1:
	        print(value)
	0
	1

Generator comprehensions are one way of creating generators, the other one is
to create the generator functions. The generator function creates (and sends
back) a sequence of results instead of a single value. This is done by using
the `yield` statement instead of `return`.

	:::python
	from math import sin

	def iterate_sine():
		current_value = 0
		while current_value < 90:
			yield sin(current_value)
			current_value += 1

	>>> for sine_value in iterate_sine(): print(sine_value)
	# Lots of output

The confusing point in the code snippet above can be how it returns the value
more than once?

Well, the idea behind generators is that once it hits the `yield` statement, it
returns that value, and pauses the execution of code. Next time the generator
is called, it starts executing the code once again.

Notice the while loop above, since the code is only _paused_ and not stopped,
it restarts in the next iteration, increments the value of `current_value`,
starts next iteration of the loop yields value and pauses.

If we had used `return` instead of `yield`, the loop would have stopped either
after returning only one value, or we'd have to collect all the values in an 
object and return that object.

	:::python
	def try_to_iterate_sine_with_return():
		current_value = 0
		while current_value < 90:
			return sin(current_value)
			# Keeping this for cosmetic purposes, it won't be hit anyway
			current_value += 1

	def iterate_sine_with_list():
		current_value = 0
		value_list = []
		while current_value < 90:
			value_list.append(sin(current_value))
			current_value += 1
		return value_list

While it won't matter if the object is small and the values can be easily
calculated but the use of generators is clearly advantageous when we have to
defer the calculation of values, or the object returned will turn out to be
huge.

For instance, reading a file which has millions of lines of text into a
list is going to have large space requirements, but reading the same file into
a generator, we can simply return one line of file at a time, taking down the
memory required by a large amount.

	:::python
	>>> file_obj = open('some_file.log')
	>>> for line in file_obj
		

