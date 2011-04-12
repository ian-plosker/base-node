This is a basic port of Dean Edward's Base javascript library for inheritance. You can find his original description [here](http://dean.edwards.name/weblog/2006/03/base/).

This port will be turned into a proper node.js module/package as soon as I get around to it, but for now to can be imported into a node project using the standard `var Base = requires('PATH/TO/base.js');`.

I'm also working on an eventual fork of this library that adds some generic type checking.

Below is the blog post/documentation that Dean Edwards published for the original library:

----------------------------------------------------------------------------------------------------------------------------------

## A Base Class for JavaScript [Inheritance](http://dean.edwards.name/weblog/2006/03/base/)

I'm an OO programmer at heart and JavaScript supports prototype based
inheritance. Unfortunatley this leads to verbose class definitions:

    
    
    function Animal(name) {};
    Animal.prototype.eat = function() {};
    Animal.prototype.say = function(message) {};
    

I want a nice base class for JavaScript OO:

  * I want to easily create classes without the `MyClass.prototype` cruft
  * I want method overriding with intuitive access to the overridden method (like Java's `super`)
  * I want to avoid calling a class' constructor function during the prototyping phase
  * I want to easily add static (class) properties and methods
  * I want to achieve the above without resorting to [global functions][12] to build prototype chains
  * I want to achieve the above without [affecting `Object.prototype`][13]

   [12]: http://www.sitepoint.com/blogs/2006/01/17/javascript-inheritance/
   [13]: http://erik.eae.net/archives/2005/06/06/22.13.54/

#### The Base Class

I've created a JavaScript class ([Base.js][14]) that I hope eases the pain of
JavaScript OO. It's a simple class and extends the `Object` object by adding
two instance methods and one class method.

   [14]: http://dean.edwards.name/base/Base.js

The `Base` class defines two instance methods:

##### extend

Call this method to extend an object with another interface:

    
    
    var object = new Base;
    object.extend({
    	value: "some data",
    	method: function() {
    		alert("Hello World!");
    	}
    });
    object.method();
    // ==> Hello World!
    

##### base

If a method has been overridden then the `base` method provides access to the
overridden method:

    
    
    var object = new Base;
    object.method = function() {
    	alert("Hello World!");
    };
    object.extend({
    	method: function() {
    		// call the "super" method
    		this.base();
    		// add some code
    		alert("Hello again!");
    	}
    });
    object.method();
    // ==> Hello World!
    // ==> Hello again!
    

You can also call the `base` method from within a constructor function.

#### Creating Classes

Classes are created by calling the `extend` method on the `Base` class:

    
    
    var Animal = Base.extend({
    	constructor: function(name) {
    		this.name = name;
    	},
    	
    	name: "",
    	
    	eat: function() {
    		this.say("Yum!");
    	},
    	
    	say: function(message) {
    		alert(this.name + ": " + message);
    	}
    });
    

All classes created in this manner will inherit the `extend` method so we can
easily subclass the `Animal` class:

    
    
    var Cat = Animal.extend({
    	eat: function(food) {
    		if (food instanceof Mouse) this.base();
    		else this.say("Yuk! I only eat mice.");
    	}
    });
    	
    var Mouse = Animal.extend();
    

#### Class Properties and Methods

A second parameter passed to the `extend` method of a class defines the class
interface:

    
    
    var Circle = Shape.extend({ // instance interface
    	constructor: function(x, y, radius) {
    		this.base(x, y);
    		this.radius = radius;
    	},
    	
    	radius: 0,
    	
    	getCircumference: function() {
    		return 2 * Circle.PI * this.radius;
    	}
    }, { // class interface
    	PI: 3.14
    });
    

Note the use of the `base` method in the constructor. This ensures that the
`Shape` constructor is also called. Some other things to note:

  * If you define a class method (_not_ an instance method) called `init` it will be automatically called when the class is created
  * Constructor functions are never called during the prototyping phase (subclassing)

#### Classes With Private Data

Some developers prefer to create classes where methods access private data:

    
    
    function Circle(radius) {
    	this.getCircumference = function() {
    		return 2 * Math.PI * radius;
    	};
    };
    

You can achieve the same result using the Base class:

    
    
    var Circle = Shape.extend({
    	constructor: function(radius) {
    		this.extend({
    			getCircumference: function() {
    				return 2 * Math.PI * radius;
    			}
    		});
    	}
    });
    

The code is slightly more verbose in this case but you get access to the
`base` method which I find incredibly useful.

#### Single Instances

I changed my mind a lot about this but finally decided to allow the creation
of single instance classes by defining a `null` constructor:

    
    
    var Math = Base.extend({
    	constructor: null,
    	PI: 3.14,
    	sqr: function(number) {
    		return number * number;
    	}
    });
    

#### Conclusion

This supersedes my [old OO framework][15] which was not as cross-browser as I
would have liked (and had other shortcomings).

   [15]: http://dean.edwards.name/common/
