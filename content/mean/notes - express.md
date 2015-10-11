Title: Introduction to Express
Date: 2015-01-15
Category: Express
Tags: express, deep-dive
Slug: introduction-to-express
Authors: Tushar Tyagi
Summary: A handy introduction to express.


## Express js:

### Installing express system wide ###

While there's a convention of installing node apps inside the project, we
can install express-generator because it automatically creates boilerplate
code for express and node based applications.

The following line does the trick

	:::bash
	>>> npm install express-generator -g

### Creating a new app in express ###

* Open the directory where the express app needs to be present, and express
again. This creates all the required code

		:::bash
		$ express
		create : .
		create : ./package.json
		...
		create : ./bin/www

		$ ls
		app.js  bin  package.json  public  routes  views



### Application
* Conventionally, the Express application is called `app` within our code, 
  instantiated by:

		:::javascript
		var express = require('express');
		var app = express();

### Middleware
* The middleware is any function which changes the way request and response
  objects are changed before, in, or after, the request and the response.
  Usually this is called the middleware stack, because the order in which
  middleware functions are defined is the order in which they are executed.

* They have the same syntax except the presence of a parameter called `next`
  which calls the next Middleware in line.

  Let's take an example, where the requests coming at the root are handled by 
  some function:
  
		:::javascript
		app.use('/', function(req, res) { 
			// handle the req/res 
		});
  
*  In order to introduce middleware, we have to add another function, which is 
  the middleware:


		:::javascript
		app.use(function(req, res, next) {
			// modify the req/res in some way
			next();
		});
    
		app.use('/', function(req, res, next) {
			// use the modified req/res
		});
    
	There are two important points to notice here:

	* The order is of paramount  importance. Only because the middleware came 
    before the route handling  function, the latter could use the modified 
    request. 

	* The parameter `next`, which is a method and is called just before the
	function returns. Unless `next` is called, the request will be left hanging.

* The middleware methods can be called before as well as after the routing.
  Again, we have to make sure that even the routing function calls the next 
  method in those situations. 
  
		:::javascript
		app.use(function(req, res, next) {
			// modify the req/res in some way
			next();
		});
    
		app.use('/', function(req, res, next) {
			// use the modified req/res
			next();
		});
	
		app.use(function(req, res, next) {
			// modify the req/res some more
		});

* Of course, just like the routing methods, even the middleware can take some 
	routes and then will only modified those requests which match the given 
	route.

	For instance, the following middleware will only modify the route starting
	with `/users`


		:::javascript
		app.use('/users', function(req, res, next) {
			// modify the req/res in some way
			next();
		});

* What we have been talking about is the app level middleware, which controls 
	the routing at the application level. In order make the application more 
	modular, and convenient, it's recommended to divide the routing based on 
	different routes.

	Let's say we have a section of the application under the route `/users`. In
	order to use it, we first have to have a separate routing file, let's call 
	it users.js and place it under `routes` directory.

	Then from the main app, we can access it using:

		:::javascript
		var user = require('./routes/user');
		app.use('/user', user);

	In the file `users.js`, we can have the routing logic:

		:::javascript
		var express = require('express');
		var router = express.Router();

		// the next route starts from / but since it's already redirected
		// from the root, it essentially means /user
		router.get('/', function(req, res) {
			// Show a welcome screen
		});

		// Path is /user/account from the main app
		router.get('/account', function(req, res) {
			// Show account screens
		});

		// Path is /user/logout from the main app
		router.get('/logout', function(req, res) {
			// Logout user
		});

	Of course, we have the option of using middlewares in the routing files 
	too.
