BaseJS
=========

#### Helper Library for Prototyping Javascript Objects ####

BaseJS provides methods for building object inheritance and
mixing object definitions to promote consistent, reusable
patterns.  The objective of this project was to learn the
different concepts related to Javascript Prototypal Inheritance
add extra functionality where there seemed to be a gap in the
built-in features.

### Features ###

- Does not depend on any other library
- Provides a root object definition Base that can be extended to create new object classes via extend() method
- Handles function collisions and creates _super(), _superApply(), and _superStop() to enable invoking parent functionality seemlessly in child implementation
- Allows additional objects to be "mixed" into the object prototype and chains function name collisions automatically


Usage
---------------

To use BaseJS in your application, either download the 
[Minified](https://raw.github.com/bseth99/basejs/master/base.min.js) or 
[Full](https://raw.github.com/bseth99/basejs/master/base.js) version and copy it to a suitable location. 
Then include it in your HTML like so:

    <script type="text/javascript" src="/path/to/base.js"></script>


### Basic ###

The library contains a starting object Base which contains two static members - extend() and mix().  To start
prototyping a new object definition, call Base.extend():


      var Sub = Base.extend({

         do_something: function() {

         }

      });

      var s = new Sub();

Base.extend() will return a function with a prototype that has the function do_something() defined.  Additionally, Sub is a child of Base and 
will also contain the static members extend() and mix() which can be used to define new objects (via extend) or further enhance Sub (via mix):

      var Sub2 = Sub.extend({

         do_more: function() {

         }

      });

      var s = new Sub2();

Now Sub2 is a child of Sub and Base.  Anything defined in Sub will also be available in Sub2.  If you want a constructor, you can either
pass it in the object hash or define it prior to calling extend:


      var Sub = Base.extend({

         constructor: function() {
            // initialize
         }

      });
   
or

      var Sub = function() {
         // initialize
      };

      Sub = Base.extend(Sub);

However, in the latter, you'd have to setup Sub.prototype outside Base.extend().  


### Overrides and Chaining ###

Whenever there is an collision between a parent and child method, the library will wrap the colliding methods and
enable a means to call the parent's method from the child method via the _super() function.  The _super() function
refers only to the immediate parent of the child object.  If more than one object is in the chain, each ancester needs
to explicitly call _super() to continue invoking the next parent in the chain.  The exception is constructor and mixin
conllisions.  Constructors will automatically chain from the child up to the top-most parent automatically.  Collisions 
with mixin functions will also chain.  If you want to stop this behavior, call _superStop() to cancel the chaining to the
next parent.

      var Sub1 = Base.extend({

         constructor: function() {
            // initialize Sub1
         },

         foo: function() {
            // Sub1.foo
         }

      });

      var Sub2 = Sub1.extend({

         constructor: function() {
            // initialize Sub2
         },

         foo: function() {
            // override Sub1.foo
            // Call Sub1.foo
            this._super();
         },

         bar: function() {
            // Sub2.bar
         }

      });


      var Sub3 = Sub2.extend({

         constructor: function() {
            // initialize Sub3 
            // Implicit chaining to all the ancestors
            // Sub3 --> Sub2 --> Sub1
            // Could call this._superStop() to prevent
            // Sub2 and Sub1 from executing
         },

         bar: function() {
            // override Sub2.bar
            // Sub2.bar will not run
         }

      });   
   

### Object Properties ###

If there are properties defined on both the child and parent objects, the child will override the 
parent on all primative types.  Objects and Arrays will try to intelligently merge them when possible
using a deep comparison method:

      var Sub1 = Base.extend({

         a_string: 'string',      
         a_object: { x: 1, y: 2 },      
         a_array: [0,1,2]

      });

      var Sub2 = Sub1.extend({

         a_string: 'string2',      
         a_object: { y: 5, z: 3 },      
         a_array: [0,2,2,4]

      });
       
The prototype for Sub2 would look like this after extending Sub1:

      a_string: 'string2',  /* child overrides */
      a_object: { x: 1, y: 5, z: 3 },  /* child overrides collisions, merges parent and child definitions */
      a_array: [0,2,2,4] /* each matching index is checked, sub-objects will be merge recursively, primatives overwritten.  Missing indexes added */

   
### Mixins ###

Additional objects variables can be defined with properties and functions to mix into an object definitions prototype.  This can be in the extend() call
or using the mix() function after defining the object with extend():

      var Mix = {

         var1: 10,
         sum: function ( s ) {
            return this.var1 + s;
         }
      };
   
This object can be mixed:

      var Sub1 = Base.extend([Mix], {

         ...

      });   

or 

      var Sub1 = Base.extend({

         ...

      });  

      Sub1.mix([Mix]);

There is a difference in the two approaches.  In the former, any collisions between Mix and Sub1 will result in a chain from Sub1 to Mix and in the latter,
Mix will be called first then Sub1.  So if both Mix and Sub1 implement a sum() function, the first case will call sum() in Sub1 first followed by sum() in
Mix.  It will call in the opposite order in the second case.

More than one object can be mixed at a time:

      var MixA = {      
         run: function () {}
      };

      var MixB = {      
         run: function () {}
      };

      var Sub1 = Base.extend([MixA, MixB], {   
         run: function () {}     
      }); 
   
Since all of the object define sum(), it will be automatically called on each object from the Sub1, then MixB, then MixA.  The order of presidence is from
right to left (child first, last mixin in the array, second to last mixin, and so on.

   
### Static Methods/Properties ###

An optional object can be provided after the prototype definition that represents static properties and/or methods to add to the object:

      var Sub1 = Base.extend({

         /* prototype properties/methods */

      }, {

         /* static properties/methods */

      }); 


You can also do this when adding mixins as well:

      var Sub1 = Base.extend([MixA, MixB], {

         /* prototype properties/methods */

      }, {

         /* static properties/methods */

      }); 




Example
---------------

This example is taken from one of the [unit test cases](https://bseth99.github.com/basejs/tests/run.html) and shows the most common usage:

      var log = [];

      var Animal = Base.extend({

         constructor: function () {
            log.push('Animal.constructor');

            this.init();
         },

         init: function () {
            log.push('Animal.init');
         },

         state: 'sleep',

         eat: function () {
            log.push('Animal.eat');

            this.state = 'eat';
         },

         sleep: function () {
            log.push('Animal.sleep');

            this.state = 'sleep';
         },

         roam: function () {
            log.push('Animal.roam');

            this.state = 'roam';

         }

      });

      var Tricks = {

         speak: function () {
            log.push('Tricks.speak');
         },

         rollover: function () {
            log.push('Tricks.rollover');
         },

         special: function () {
            log.push('Tricks.special');
         }

      };

      var Habits = {

         chaseTail: function () {
            log.push('Habits.chaseTail');
         },

         special: function () {
            log.push('Habits.special');
         }

      };

      var Dog = Animal.extend([Habits, Tricks], {

         eat: function () {
            this._super();
            log.push('Dog.eat');
         },

         sleep: function () {
            log.push('Dog.sleep');
            this._super();
         },

         special: function () {
            log.push('Dog.special');
         }

      });

      var a = new Dog();

      a.eat();
      a.sleep();
      a.roam();
      a.rollover();
      a.speak();
      a.chaseTail();
      a.special();

      console.log( log );

