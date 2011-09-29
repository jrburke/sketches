# OrderedObject

A sketch to see if an **Ordered Object** construct could help with:

* Class construction syntax, something that is declarative.
* A syntax that would support default function argument values
* Declarative module syntax that maps closely with Class syntax.

I am lover of the JS language, but not a language designer. I use the language
a lot to build web applications, so this feedback comes from a language user
not a someone who designs languages.

## Basic definition

An ordered object is a hybrid of an array and an object. Functions support
ordered arguments, but those arguments are named, and it would be useful to
allow default values for some arguments.

The bracing syntax is open to discussion, but since it is a combination of an
arryay and an object, this syntax will be used for this document:

    {[ ]}

There are some advantages to this syntax:

* Similar to object, uses array brackets so implies an ordering too.
* Even though it is two characters for each segment, the two characters are on
  the same key.
* It seems different enough to not clash with existing token ordering?

There is the possibility of being more extreme and converting existing object
syntax as just be ordered object syntax, but that may be bridge too far. I
am open to it though, helps cut down on typing makes for a more consistent
idiom.

However, the rest of the document will use {[]} so it is clear this
new ordered object form is in play.

Items in the ordered object are listed similar to object key: value syntax,
but the order is preserved, and access to the values in the object can also
be done by index:

    var obj = {[
        name: 'foo',
        size,
        10
    ]};

In this example, an object is created that has the following properties:

    obj.name   //'foo'
    obj.size   //undefined
    obj[0]     //'foo'
    obj[1]     //undefined
    obj[2]     //10
    obj.length //3

There are some "reserved words" for property names, to support things like
getting the length of the keys. While this may seem a bit odd, I think it just
means considering the items in the list like keywords in a language.

List of reserved words, cannot be used for user-space
activities, but have special uses.

* length (does not count properties )
* super
* constructor
* call


## Function usage

Ordered objects can be used to create function definitions for their arguments,
and the names of the ordered object properties are automatically reflected
in the function body as function arguments, and it serves as the basis for
the **arguments** object

    function execute({[
        name: 'foo',
        quantity,
        color: 'blue'
    ]}) {
        if (quantity > 3) {
            color = 'black';
        }

        return arguments.length;
    }

    var result = execute('bar'); //returns 1, arguments.length is 1.


The funciton execute({[]}) might be a bit too much line noise, the form may be
able to be simplified. Maybe:

    function execute {[
        //arguments
    ]} {
        //function body
    }

The following may be an option too, but there may be symmetry/consistency
reasons to always use {[]} to denote ordered object use:

    function execute ({
        //arguments
    }) {
        //function body
    }

## Class usage

An ordered object is used to set up the instance properties of a new object,
when used as part of the special **constructor** function in a Class syntax.

Further info on the Class syntax used below:

* Still prototype-based, this is just sugar.
* An ordered object is used to declare the class. The properties in that ordered
object become prototype properties. The ordered object in the class
definition is the prototype object.



xxxx
* All identifiers inside a class default to **this.identifier**, so no need to
use "this" all the time inside a class.


* To disambiguate with a function that has a named argument that is the same
as an instance variable, use arguments.identifier since arguments will be
an ordered object.

    class Person {[
        //These are all prototype properties
        legs: 2,

        hop: function () {},

        run: function () {
            if (legs > 2) {
                //Do something fancy
            } else {
                hop();
            }
        },

        speak: function (message) {
            console.log(message);

xxxx resolution of console


        },

        //Special, see reserved words near the top of the document.
        //The constructor function called via *new*
        constructor({[
            name,
            gender: 'female'
        ]}) {
            speak('hello, my name is ' + name);
        },

        //Special: call, allows creating an instance that is
        //a callable function, and when called in this way, this
        //function executes.

    ]}

    //Class methods bound via a mixin function. These are less
    //common so this is OK. The point of classes is to get proptotype
    //reuse. mixin to be specified more, but basically, copy the enumerable
    //properties of the second arg onto the first arg. This could even be
    //user-space defined.
    mixin(Foo, {
       classMethod1: function (){},
       classMethod2: function (){}
    });


## Module usage
