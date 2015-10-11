Title: Decorators in Python
Date: 2015-01-15
Category: Python
Tags: decorators, deep-dive
Slug: decorators-in-python
Authors: Tushar Tyagi
Summary: Decorators in Python

## Why do we use functions? ##
Functions were introduced in programming to reduce clutter and to improve
reusability and readability. Any piece of code which

* repeated itself multiple time
* had an independent context

could be taken away and recreated as a function. This function was then called
wherever that piece of code was required.

As the code grows, we may have some code inside a function which is repeated,
and has a meaning independent of the parent function. We can again take that
piece of code outside the function, and create another function out of it.

## And Decorators? ##

Decorators follow somewhat same coding pattern. They are constructs that allow
us to change the way function behaves at runtime, and add new features to it
which are otherwise independent of that function.

Let's take a function which formats and prints the contents of a logfile. For
some cases, you also want to add security to it so that only the valid users
can execute it. What do you do? You go ahead and change the body of the
function, adding security inside of it.

Then after some days, you need to add the same security logic to the login
module of your application. While we're at it, let's add the same security
logic to the file manager of the app.
sssss
Does it make sense to open up the existing code and add some reusable
functionality which is independent of the module logic? This is where
decorators shine.

Decorators allow us to enhance or entirely change the logic of a function
at runtime, without changing the body of a function. 


We use functions in order to avoid repetitive code, i.e. any piece of code which
repeats itself multiple times is hidden behind a function. Using the same logic
if we have some code that is a part of multiple functions, and is independent
of the function's core behavior, can be hidden in a decorator. During runtime,
the decorator modifies the behavior of the function, without modifying
function's body.


### Useful Concepts ###

Before jumping into the code of decorators, let's check out what is already
present for us.

#### Functions as Objects ####

Python provides the functionality of treating functions as first class objects,
so anything that can be done with objects can be done with functions.
We can

a. Pass them around

	:::python
	def f():
		print("foo")

	def execute_function(f):
		f()

	>>> execute_function(f)
	foo

b. Store them in tuples/lists

	:::python
	def f():
		print("foo")

	def g():
		print("bar")
	
	def h():
		print("baz")

	>>> fun_tuple = (f, g, h)
	>>> for function in fun_tuple:
		    function()
	foo
	bar
	baz

c. Add attributes

	:::python
	def f():
		print("foo")

	f.__author__ = 'Me'

	>>> print(f.__author__)
	Me


#### Inner functions and closures ####
Functions can also contain other functions and these inner functions have
access to the data of the parent function, even after the outer function is
no longer present in the scope.

	:::python
	def outer(arg_outer):
		outer_data = "some data"
		def inner(arg_inner):
			print("arg_inner: ", arg_inner)
			print("arg_outer: ", arg_outer)
			print("arg_outer: ", outer_data)
		inner(12)

	>>> outer(21)
	12
	21
	some data

### Syntax ###
Decorators are just functions which take a function, modify and return it.
__Returning a function__ is an important aspect of decorating the function.
Whatever a decorator returns has to be callable, it can either be another
function, or a class which implements `__call__` method.

The syntax is straight forward:

	:::python
	def deco(func):
		# Let new_func = do something to func
		return new_func

And we can use it like this:

	:::python
	@deco
	def some_func(*args, **kwargs):
		pass

And behind the scenes, this happens:

	:::python
	some_func = deco(some_func)

So the original function is replace with the modified function.

### Some examples which make sense ###
Let's just say that you have some very expensive functions which are
finding solutions to some of the greatest problems ever encountered by mankind.
These aren't called just like that since they're expensive computationally, and
hence we'll have to log each one of them.

One way of doing that is to add the logging statements inside each of the
functions, something like:

	:::python
	def expensive_func_1():
		print("expensive_func_1 called at ", time.strftime("%H:%M:%S"))
		# do something expensive

	def expensive_func_2():
		print("expensive_func_2 called at ", time.strftime("%H:%M:%S"))
		# do something expensive

Now adding something like this has multiple bad things.
* We are going against the practice of DRY (Don't repeat yourself)
* The logging isn't required by the function to do its job -- it's an addon.
So it shouldn't be part of it.

So we sense that it's repeated code, and repeated code needs to have a function
for itself. But function inside function?? Yes that's possible and we can have
something like:

	:::python
	def log_it(func_name):
		print(func_name, " called at ", time.strftime("%H:%M:%S"))

	def expensive_func_1():
		log_it(expensive_func_1.__name__)
		# do something expensive

	def expensive_func_2():
		log_it(expensive_func_2.__name__)
		# do something expensive
	
But again, we are including something inside a function that shouldn't be part
of it. It's code inside a function that repeats itself, but isn't really needed
inside it. So in situations like these, we can start using generators.

	:::python
	def log_it(func):
		def new_func():
			print(func.__name__, "started at ", time.strf_time("%H:%M:%S"))
			func()
		return new_func

What happened? We just created a decorator.
Like I mentioned before, a decorator is a function which changes another
function, in a way enhances it's working. So this generator takes in an
expensive function and returns another function which

a. print the log time
b. calls expensive function.

And how's it different from the previous approach?
Because it's nicer to look, for starters. This is how we use it:

	:::python
	@log_it
	def expensive_func_1():
		print("Running expensive_func_1")

	@log_it
	def expensive_func_2():
		print("Running expensive_func_2")


	>>> expensive_func_1()
	expensive_func_1 started at 21:36:03
	Running expensive_func_1

	>>> expensive_func_2()
	expensive_func_2 started at 22:36:03 # It's expensive
	Running expensive_func_2


While this doesn't seem much at such small scale, the most important point in 
the favor of decorators is that they are independent of function body. So, 
let's say you have to do some extra logging in one of the functions; with the
previous approach, we'll have to change the name of the function call inside
the expensive function, like:

	:::python
	def expensive_func_2():
		better_log_it(expensive_func_2.__name__)
		# do something expensive
	
With generators, we don't have to touch the body of the expensive function:

	:::python
	@better_log_it
	def expensive_func_2():
		# do something expensive
	
Still not impressed? Let's take the thing a bit further:

#### Decorator chaining ####
Decorator syntax is influenced by mathematical functions. In mathematics,
the statement: `h(g(f(x)))` means that function g will take the the output of
f, and function h will take use to the output of g for the calculation.

You can chain decorators as well:

	:::python
	@h
	@g
	def f(x):
		# do something with x

This modifies the function f like this:

	f = h(g(f(x)))

### Passing arguments in decorators ###
Since decorator is a function, we can pass arguments to it as well. These
arguments can then go and change the way a decorator behaves and hence change
the way it decorates the function.

Uptil now, we were using decorators without any arguments, and the syntax was:

	:::python
	@deco_without_args
	def func(farg1, farg2):
		pass
	
And the decorator body was returning a callable, either a function or a class
which implemented __call__ method. I'll repeat the code here once again to be
more clear:

	:::python
	def deco_without_args(func)
		def wrapped_func(*args, **kwargs):
			# do something with func, possibly func(*args, **kwargs
		return wrapped_func

The syntax of decorator with arguments is:

	:::python
	@deco_with_args(darg1, darg2, darg3)
	def func(farg1, farg2):
		pass

And since the decorator is being called here (it's clear from the () operator),
the decorator call itself should return a callable. It's another way of saying
that since decorator is a function, this syntax means that the function call
should return another function. How can we achieve that? Like this:

	:::python
	def deco_with_args(darg1, darg2, darg3):
		def actual_decorator(func):
			def wrapped_func(*args, **kwargs):
				# do something with func, possibly func(*args, **kwargs)
			return wrapped_func
		return actual_decorator

Since both the argument list of darg* and the decoratee function func are
accessible inside wrapped_func, we can use them to modify the function in
whatever ways we desire.

Let's take the case wherein we want the logging decorator to be able to can
print the date and time based on some specific format. We will pass the format
as argument of the decorator.

	:::python
	def format_and_log(fmt_string):
		def deco(func):
			def new_func():
				print(func.__name__, "started at ", time.strf_time(fmt_string))
				func()
			return new_func
		return deco


	@format_and_log("%d/%m/%y %H:%M:%S")
	def expensive_func_3():
		print("Expensive function 3")

	>>> expensive_func_3()
	expensive_func_3 started at  20/02/15 11:47:30
	Expensive3 function 3


With this, the major ideas with respect to decorators are covered. The
possibilities of using decorators to create reusable code, and removing the
clutter from functions are there. 
