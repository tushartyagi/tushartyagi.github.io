Title: Object Oriented JavaScript
Date: 2015-03-10
Category: javascript
Tags: javascript, oojs
Slug: object-oriented-javascript
Authors: Tushar Tyagi
Summary: Implementing Object Oriented Javascript

# Object Oriented Javascript #

JavaScript doesn't support class based oo programming, it has object based 
oo support. This means that instead of creating classes and instantiating them 
we simply start with objects and an object can inherit attributes and 
properties from other objects.

## A Digression ##
Before jumping into actual OO code, I want to  digress and show how inheritance
works in JS.

In the developer tool of any browser, if I create a new object, and checkout
it's properties, there's a property called `__proto__`. This is the heart of
JS inheritance structure.

	:::javascript
	>> var func = new Function();
	>> func.__proto__; // function ( )

	>> var obj = new Object();
	>> obj.__proto__; // Object ( )

	>> var arr = new Array();
	>> arr.__proto__; // Array [ ]

Now this is a hidden property and cannot be accessed in our code. But it's
important. Here's how:

Since there's no concept of class in JS, an object has to depend on itself for
properties. If it doesn't have a property, it looks into its parent object
through `__proto__`. If the property is still not found, the grandparent is
checked for the property and this goes on till we reach `Object`. Now, `Object`
has this property as `null` so if we still can't find the property, an
exception is thrown. 

Following the same logic, we see that `func` will have function properties,
like `call` and `apply`; `arr` will have `push()` and `pop()` etc.

## Create A Class ##

One look at the statements above and you'll notice that I'm  calling some
constructors: `Function`, `Object`, and `Array`. These constructors create
a new object and initialise it's `__proto__` property.

By convention, the names of constructors are capitalised and these are
always used with `new` keyword, something like:

	:::javascript
	var obj = new Cons(); 

So there's no way of creating a class in JS, we just have to create
constructor functions and create our objects using these.

Inside the constructors, we have access to `this` keyword, which works a lot
like other OO languages.


	:::javascript
	function Person(firstName, lastName) {
		this.firstName = firstName;
		this.lastName = lastName;

	    this.getFullName = function() {
			return this.firstName + " " + this.lastName;
		}
	}

	var peter = new Person("Peter", "Parker");
	peter.getFullName(); // Peter Parker

## Another prototype?? ##

If I look at Person() in developer tools, I'll see another property called
`prototype`. And it's only present in function objects, not in `Object`s and
`Array`s. What is this thing?

Before I answer that question, let's see how the dynamic duo of constructors
and `new` really work behind the scenes.

	:::javascript
	var bruce = new Person("Bruce", "Wayne");

1. A new, empty object is created 
2. This object's `__proto__` is set to Person.prototype
3. The constructor function is called, with `this` keyword set to the new
object.
4. The reference of this new object is stored in `bruce`.

The second point is really important to understand what `prototype` does and
why it's used. 

An object's `prototype` will hold all the properties and values which will be
used by every object that instantiates from it. If you look at the second point
`__proto__` is used for property lookups, and if we point it to some object, we
have an upper hand as to where to start looking for properties.

I'll explain this again so as to drill it further down. The `prototype` object
is just another object containing some properties and methods. When we create
an object its `__proto__` gets a reference to `prototype` object of the
constructor and hence it gets all the goodies inside `prototype`. Whenever we
need a property, we check the object and if not found, go searching through
all the `__proto__`s all the way to the object.

Now if you want to ask where is `Person.prototype` is pointing to? It's
pointing to the parent of all objects: Object. Every top level constructor's
prototype refers to `Object`.

TL;DR: `__proto__` is checked while trying to find the properties through the
parents and grandparents. `prototype` is simply used to hold a reference to
the constructor with which we've created a particular object.

And since `prototype` is writable, we'll change it to our advantage in the
by adding properties which are used by all the objects sharing the same
prototype.

	:::javascript
	Person.prototype.walk = function() { console.log("walking"); }
	bruce.walk(); // walking
	peter.walk(); // walking

The effect is dynamic and updates older objects as well. If we check
`Person.prototype` in console, it now contains `walk()`.

## Extend Already!! ##

Another main concept of OOP is that a child class can inherit properties and
methods of its parent class.

Now we know that `prototype` is writable, and contains the properties which
will be inherited by child objects.

Let's create a superhero which will inherit from `Person`. There are some
steps that we have to follow in order to extend an object.

1. Create the child object

		:::javascript
		function Superhero(firstName, lastName, power) {
			// call to "super" or parent constructor
			Person.call(this, firstName, lastName);
			this.power = power;
		
			this.showPower = function() {
				console.log(this.power);
			}
		}

	But check `Superhero.prototype` here, it's still `Object` and doesn't
	contain the methods we added to `Person`, so we need to change the
	`prototype`.

2. Change the prototype of child object 

		:::javascript
		// This means that the Superhero will copy all the
		// properties of Person object
		Superhero.prototype = Object.create(Person.prototype);

	Now checkout the `Superhero.prototype.constructor`, it is `Person`. This
	attribute lets us know what constructor do we have to call when `Superhero`
	is called. We know that it has to be Superhero, so we have to change this
	as well.

3. Let the Superhero know that it's a Superhero

		:::javascript
		Superhero.prototype.constructor = Superhero;

4. Execute

		:::javascript
		var spidey = Superhero("Amazing", "Spiderman", "Spins web");
		spidey.getFullName(); // Amazing Spiderman
		spidey.walk(); // walking
		spidey.showPower(); // Spins web

## Parting words ##

I hope this article was helpful in painting a clearer picture of OOP in
JavaScript. Of course, we cannot have everything which mainsteam classical
languages provide, something like inheriting private variables comes to mind.
