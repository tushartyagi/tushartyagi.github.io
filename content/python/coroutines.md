Title:  Coroutines in Python 
Date: 2015-01-15
Category: Python
Tags: coroutines, deep-dive
Slug: coroutines-in-python
Authors: Tushar Tyagi
Summary: Coroutines in Python

Coroutines have always seemed to be an interesting concept to me, although I've
never dived deep in that concept. When I first heard the name while trying to
learn Lua (which I didn't pursue), I thought this was just another fancy
name of subroutines.

Before going further, we have to notice the similarities and differences
between the concepts of subroutines and coroutines.

Subroutines are the normal functions that we have, they may or may not return
a value. Coroutines are a lot like subroutines, but instead of returning values
they _yield_ values.

A subroutine works in context to a parent function. So the parent code can be
made up of multiple subroutines, with each one executing, doing its work, and
getting destroyed once the work is complete. If the parent calls again, a new
instance of subroutine is created and executed which is independent of the
previous call.

In contrast, a coroutine can start executing, do its work, pause, and  return
to the parent. When the parent calls again, the coroutine starts from where it
paused during the previous call.

The best real world application I could find was using Coroutines for
implementing finite state machines, since these too store their current state
and move to another state based on inputs, and resume execution once the
next state has finished its job.

Before implementing finite state machines using coroutines, let's start with
the syntax.

### Syntax of Coroutines ###
Coroutine derive its syntax from generator because both of these work on the
concept of pausing and yielding values.

As we've seen in Generator functions, we can either return a value which
completes the execution of a function, or we can yield a value which
temporarily suspends the execution until the function is called again.
Coroutines work the same way, though a bit in reverse than generators --
generators provide values, coroutines consume values and do something with them.
While a piece of code can be written in a way so that it produces and consumes
values at the same time, [it's not a good idea to do so](
http://www.dabeaz.com/coroutines/Coroutines.pdf)

Generator functions return value using a `yield` statement:

	:::python
	def gen():
		yield 1
		yield 2
		yield 3

	>>> gen()
	1
	>>> gen()
	2
	>>> gen()
	3

	def a_coroutine():
		some_value = (yield)
		# do something with some_value

	a_coroutine.send(value)
	
Coroutines use the same yield statement to consume values, like this:

	:::python
	some_value = (yield)

When, inside the function body, a yield statement is hit, the execution pauses
and the function waits for some value from the caller coroutine, which uses
`coroutine.send()` to send the value.

	:::python
	coroutine.send(value)

When `send` is called, the execution starts again, the caller does its work
and pauses again when the next `yield` statement is hit. This goes again until
the caller calls the `close` method which  raises `GeneratorExit` exception
inside the callee, which we have to catch and exit gracefully.

	:::python
	coroutine.close()

Using the above concepts, the following code provides a concrete example of
coroutines:

	:::python
	# Find a substring when strings are passed
	def coroutine_substring(substring):
		print("Waiting for strings")
		try:
			while True:
				whole_string = (yield)
				if substring in whole_string:
					print(whole_string)
		except GeneratorExit:
			print("Exiting")

A coroutine, once instanciated will need to be primed by calling its __next__()
method, this executes the code till the first `yield` statement. It can then
start functioning as required:

	:::python
	>>> co = coroutine_substring("python")
	>>> co.__next__()
	Waiting for strings
	>>> co.send("I see a python")
	I see a python
	>>> co.send("I don't see anything")
	>>> co.send("There it is!")
	>>> co.send("Can't you see the python")
	Can't you see the python
	>>> co.close()
	Exiting
	
### Coroutines as pipelines of data ###
Since coroutines can call other coroutines, these can be used as pipelines
which pass around the data and do something with it.
Conceptually, each coroutine will require data on which to act upon. This data
is sent by either another coroutine or by some source function.

As a trivial example, let's read a file and count the number of vowels in lines
which contain the word 'gene' in the text.

	:::python
	def source(filepath, process):
		file_obj = open(filepath)
		for line in file_obj:
			process.send(line)
		process.close()

	def co_filter_and_count(substring, co_next):
		try:
			while True:
				data = (yield)
				if substring in data:
					co_next.send(data)
	        co_next.close()
		except GeneratorExit:
			print("Exiting co_filter_and_count")

	def co_count_vowels():
		vowels = list("aeiou")
		try:
			while True:
				data = (yield)
				vowel_count = 0
				for char in data:
					if char in vowels:
						vowel_count += 1
	            print(vowel_count)
	    except GeneratorExit:
			print("Exiting co_count_vowels")

	# Prime the coroutines before using them
	>>> cv = co_count_vowels()
	>>> cv.__next__()
	>>> fnc = co_filter_and_count()
	>>> fnc.__next__()
	>>> filepath = "/home/user/texts/selfish_gene.txt"
	>>> source(filepath, fnc)
	# Lots of output

### Coroutines as finite state machines ###



Refences: <br>
1: http://wla.berkeley.edu/~cs61a/fa11/lectures/streams.html <br>
2: http://www.dabeaz.com/coroutines/Coroutines.pdf <br>
3: http://c2.com/cgi/wiki?CoRoutine <br>
4: http://en.wikipedia.org/wiki/Coroutine <br>





