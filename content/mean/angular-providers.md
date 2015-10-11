Title: Angular Providers
Date: 2015-03-21
Category: AngularJS
Tags: angular
Slug: angular-providers
Authors: Tushar Tyagi
Summary: What is Angular provider, and how it exposes different services, and how each of these are connected to the other.

# Angular Providers

### A digression into Services 
Before we start with what providers are, we need to understand what is meant
by Service. According to AngularJS docs, it's *a substitutable object which
wires the application together and work through dependency injection.*
Substitutable here also means that it is injectable through dependency
injection. 

That sums it up pretty nicely. Anytime you need to provide some sort of
"service", with API which you've defined, you need to create a service. In
contrast, Directives, Controllers, Filters use the API provided by Angular,
with Service you can provide your own API.

We use the `$provide` service to create new services. `$provide` provides
multiple recipes -- which are just the structure we follow when creating our
own service. Notice that while the structure of creating recipes is fixed, 
the API you open up in these recipes depend entirely on you.

The recipes (in the order I'll explain them) are:

1. value
2. factory
3. constant
4. provider
5. service 

### The Confusing Names!!
If you notice above, we are using `$provide` service to create different
service recipes. One of the recipes is called `provider` and another is called
`service`. But `provider` isn't a provider; both of these are service
recipes. What is happening here?

We first need to understand that providers and services are two different
concepts. Angular itself provides some services for core functionality, apart
from `$provide`, some examples are `$http`, `$route`, `$resource`, etc. Each
service (as noted above) exposes a different api to provide its functionality.

Every provider recipe also gets a serviceProvider, which is
"serviceName + Provider". So the above services will have `$httpProvider`,
`$routeProvider`, etc.

The relationship between these two is that the serviceProvider is used to
configure the service instances, during the bootstrap, more on that later.

I'll be using the following html, just plug it into the body and include
angular and script.js file

	:::html
    <div ng-app="app" ng-controller="MyController">
      <input type="text" ng-model="distance"/>
      <button ng-click="calcTime()">Calculate Distance</button>
      <div>The time required is: {{ spaceTime }}</div>
	</div>

Let's start with the recipes.

### value recipe
The first one is value recipe, which allows you to create a key value mapping.
The value can be a valid javascript object.

	:::javascript
    var app = angular.module("app", []);
    app.value("speedOfLight", 3.8E8);

The drawback of using a value recipe is that it doesn't allow dependencies, it
is only allowed to hold module wide "static" values which don't change. The
speed of light is one such example. 

The factory, provider, and service recipes mitigate this drawback.

### factory recipe
The second recipe is factory recipe and is created using `factory` function. 

	:::javascript
    app.factory("timeCalcFactory", ["speedOfLight", function(c) {
        return function(distance) {
            var time = distance / c;
            return time;
        };
    });
    app.controller("MyController", ["$scope", "spaceTimeCalc", function($scope, spaceTimeCalc) {
        $scope.distance = 10;
        // use calcTime with ng-click
        $scope.calcTime = function() { 
            $scope.spaceTime = spaceTimeCalc($scope.distance);
        };
    }]);

A few takeaway by the factory code:

1. It can depend on the values from other service recipes, `speedOfLight` in
the previous code is value recipe, but factory can take other factories,
providers, services, etc.
2. It has to return a function or an object which provides the functionality,
and it is this returned value which is injected. What to return will depend on
the use case, if it has to be callable, return a function, if you want multiple
properties then return an object and access them from this object.

The following code converts the previous factory to one which returns an
object. The controller has been changed accordingly: 

	:::javascript
    app.factory('spaceTimeCalc', ['speedOfLight', function(c) {
        return {
            getTime:  function(distance) {
                var time =  distance / c;
                return time;
            },
        };
    }]);
    app.controller("MyController", ["$scope", "spaceTimeCalc", function($scope, spaceTimeCalc) {
        $scope.distance = 10;
        // use calcTime with ng-click
        $scope.calcTime = function() { 
            $scope.spaceTime = spaceTimeCalc.getTime($scope.distance);
        };
    }]);

### Caching

One thing to keep in mind is that the value returned by the recipes are cached
the moment recipe is called. It is only this cached "first value" which is
returned on subsequent calls to the recipe.

### A digression: phases in angular

We are soon going to discuss constant and provider recipe, but before that
we'll see how the angular compiles and runs the application by dividing it
into two phases: `config` and `run`.

The config phase is used to do the application wide configuration, and allows
only access to provider and constant recipes.
So for example, `ngRoute` is used for application level routing and hence it
is configured here. Other service recipes aren't allowed to run in this phase,
because there's a chance that their dependencies have not been initialised.

The run phase comes after the config phase and it is during this time the
application starts to initialise services, factories, values, directives,
controllers, etc. providers and constants are not allowed to run during this
phase.

### constant recipe
constant recipe is similar to value recipe, but as noted above it isn't allowed
to run during the `run` phase. So in order to provide "static" values to
`provider` we can use `constant`. The above `value` recipe 
can be converted to `constant` like this (be sure to remove the value if you're
using the same file):

	:::javascript
    app.constant('speedOfLight', 3.8E8);

### provider recipe
The `provider` recipe is the most versatile of all the recipes and provides the
syntactic sugar for other recipes. If we check out the [source](https://github.com/angular/angular.js/blob/master/src/auto/injector.js#L665)
, we see that `value` and `service` internally call `factory`, which
internally calls `provider`. 

In order to work, the `provider` needs you to the implement a method called
`$get`. When `$get` is called, the provider should return a factory object
(which is cached and returned on further calls to provider). 

Implementing the above factory code as provider, we see that the property
`$get` is a method which returns the factory function. This is an important
point, whatever we have written as the factory function doesn't become
the `$get` method; that function is returned when `$get` is called. Of course,
if we would have implemented with the second approach above, `$get` would have
returned an object containing `getTime` method.

	:::javascript
	// The provider depends on the value of speedOfLight constant
	app.provider('spaceTimeCalc', ['speedOfLight', function(speedOfLight) {
        this.$get = function() {
			// The factory function we had before is returned when the $get 
			// method is called. 
            return function(distance) {
            var time =	distance / speedOfLight;
            console.log("The time required is: ", time);
                return time;
            };
        };
    }]);

So whenever `$get` is called, it has to return the service instance, which is
cached and used on subsequent calls.

Using this code, we don't have to make changes to our controller. If we join
the code above and below, we come to realise that `spaceTimeCalc` below is the
function returned by `$get` above and hence takes in the distance parameter
and use it in some constructive way.

	app.controller("MyController", ["$scope", "spaceTimeCalc", function($scope, spaceTimeCalc) {
        $scope.distance = 10;
        $scope.calcTime = function() { 
			$scope.spaceTime = spaceTimeCalc($scope.distance);
        };
    }]);

### Why all this ho-haa for?
The only question now remains (apart from the discussion of service recipe),
is why to use providers at all? As per the
[docs](https://docs.angularjs.org/guide/providers),

<blockquote>You should use the Provider recipe only when you want to expose
an API for application-wide configuration that must be made before the
application starts.</blockquote> 
<br/>
*Before the application starts* is the key thing here: we're talking about
`config` phase. If we want to configure the provider in some application
wide way, then we should be using providers. 

Let's say that due to some glitches in scientific studies, we've come to
realise that the speed of life isn't as constant as we think it is, and now
depends on the fraction of "ho-haa" value. So our application has to take the
correct value whenever it starts. Now we have to change the provider, and we'll
expose and API for application wide configuration:

We'll make the necessary changes in the provider, just to expose some API:

	:::javascript
	// A provider we can configure
	app.provider('spaceTimeCalc', ['speedOfLight', function(speedOfLight) {
        // mitigate evil this
        var self = this;
		self.hoHaa = 1;
        self.$get = function() {
			return function(distance) {
				var time = distance / (speedOfLight * self.hoHaa);
                console.log("The time required is: ", time);
				return time;
            };
        };
        self.setHoHaa = function(hoHaa)
		{
            self.hoHaa = hoHaa;
        };
    }]);

The `setHoHaa` method allows us to make changes in the values of provider. Now
we just need to call it before the application starts running, and for that
we'll use `config` method.

	:::javascript
    app.config(function(spaceTimeCalcProvider) {
		spaceTimeCalcProvider.setHoHaa(0.5);
    });

I've used the name `spaceTimeCalcProvider` in there, that's because we're
configuring the provider, and not the service that it provides. 

If we now run the application, we'll get double the values that what
we were getting before. 

### service recipe ###

The last recipe for creating a service is called `service`. The naming isn't up
to the mark here, with so many confusing terms. Anyways, the service recipe is
used when we use a constructor function for our object creation. 

Let's convert the above code to a constructor function, which depends on some
other services to work correctly. These dependencies are passed as parameters
of the constructor. Since we have only one, `speedOfLight`, that becomes the
parameter.

	:::javascript
	// The dependency goes into the params, there can be multiple dependencies
	var DistanceCalc = function(speedOfLight) {
		// This method uses other service, which has already been injected
		this.getTime = function(distance) {
			var time = distance / speedOfLight;
			console.log("The time required is:  ", time);
			return time;
		};
	}


If you noticed, I wrote above that `service` internally calls `factory` (which
again calls provider) to run. So we'll first look at the factory syntax of
using a constructor.

Going ahead with the logic, that the object/function returned by factory call
is cached and used, we get:

	:::javascript
	app.factory('spaceTimeCalc', ['speedOfLight', function(speedOfLight) {
		return new DistanceCalc(speedOfLight);
	}]);

Every new call to spaceTimeCalc won't create a new object; the first one
created will be reused again and again.

There's a shortcut to the above syntax, which is called the `service` recipe:

	:::javascript
	// The syntax isn't a function, it's list of dependencies
	// followed by the constructor function
	app.service('spaceTimeCalc', ['speedOfLight', DistanceCalc]);

Keep in mind the last array element isn't a function like the other recipes.

The corresponding controller isn't much different:

	:::javascript
	app.controller("MyController", ["$scope", "spaceTimeCalc", function($scope, spaceTimeCalc) {
		$scope.distance = 10;
		$scope.calcTime = function() { 
			$scope.spaceTime = spaceTimeCalc.getTime($scope.distance);
		};
	}]);


### Parting Words ###
This sums up how we can use the different recipes in our apps. While the
examples are minimal at best, I hope these do a sufficient job of explaining
how to create services, when to choose which and how all of these are
interconnected to each other.

