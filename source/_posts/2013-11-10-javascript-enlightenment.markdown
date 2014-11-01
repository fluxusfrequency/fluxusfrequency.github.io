---
layout: post
title: "JavaScript Enlightenment"
date: 2013-11-10 21:30
---

I read [JavaScript Enlightenment](http://www.javascriptenlightenment.com/) by Cory Lindley over the weekend. It was the perfect thing for me right now, as an advanced beginner. It really breaks down all the parts of tha langauge in an understandable way. I feel like I get the 'big picture' much better after having read it. Here are some notes I took along the way.

# Explanations

## Objects

In javascript, object is king.
An object is a set of properties with keys and values.

```javascript
var myObject = new Object();
myObject.name = “Ben”;
myObject.age = 29;
```

## Javascript’s Native Object Constructors

These are the nine object constructors built into JavaScript:

- Number()
- String()
- Boolean()
- Object()
- Array()
- Function()
- Date()
- RegExp()
- Error()

There’s also Math, which is not a constructor. It acts like an instantiated object.

Thus, you can do:

```javascript
Math.PI;
```

But NOT:

```javascript
var myMath = Math.new();
```

## Contructors & Instantiating With Constructors

Every object can be built with a constructor. You instantiate the object with the ‘new’ keyword.

Here’s the syntax:

```javascript
var myObject = new Object();
```

You can also define your own constructors, like this:

```javascript
var Person = function(name,age) {
     this.name = name;
     this.age = age;
}

// People can be instantiated like this:
var Ben = new Person(“Ben”, 29);
```

## Object Literal Notation

Instead of using the new keyword, you can create an object with the literal syntax:

```javascript
*var myNumber = 23;
*var myString = “Hello, world!”;
*var myBoolean = true;
var myObject = {};
var myArray = [];
var myFunction = function(){};
var myRegExp = /.*/;
var myError = (‘crap!’);
```

\* These tjree literals do NOT return an object, but a Javascript primitive. Their constructors DO return objects.
A primitive is an irreducible value.
If they are later treated as an object, for example by calling a method on them, they are temporarily wrapped in an object.

## The Third Notation

Another way to create a primitive (does not use the new keyword):

```javascript
var myNumber = Number(23);
var myString = String('foo');
var myBoolean = Boolean(false);
```


## Complex Objects

Objects that contain multiple values, which can be primitives or other complex objects.
These are objects created created with the new keyword, or literal notation (and are NOT one of the primitives).

They are also known as “composite objects” or a “reference types”.
These objects store an address reference to an object.
If you copy them, the copies are also references to the same address.
Therefore, they will all mutate if the object is changed.

```javascript
var myVarC = {name: “Ben”};
var myVarD = myVarC;
myVarC.name = “John”;
console.log(myVarD.name);
=> “John"
```


## Complex Objects - Native Properties

### 'constructor'

In addition to their basic property keys, all instantiated objects have a property called 'constructor'.

```javascript
var myVarE = {age: 29};
console.log(myVarE.constructor);
=> Object
```

### 'instanceof'

The type of an instantiated object can be checked with 'instanceof'.

```javascript
var MyVarF = {gender: “male”};
console.log(myVarF instanceof Object();)
=> True
```

### Instance Properties

Instances of a constructor can be given their own properties. This includes objects created from the nine native constructors, but does not include primitives.

```javascript
var myVarG = new Boolean();
myVarG.prop = “test”;
console.log(myVarG.prop);
=> “test”
```

```javascript
var myVarH = “hello”;
myVarH.prop = “world;
console.log(myVarH.prop);
=> undefined;
```

### Accessing Properties

Dot notation vs. Bracket notation:

```
myVarJ.prop1 = “test”
myVarJ[‘prop2’] = “bar”
```

Bracket notation has two advantages: it can be used to concatenate strings in the property name, and it can use reserved words.

```javascript
// Dot Notation
var str 1 = “foo”;
var str2 = “bar”;
var myVarK[str1 + str2] = “baz”;
myVarK.foobar;
=>”baz"

// Bracket Notation
var myVarL['class'] = “Algebra”;
myVarL['class'];
=> “Algebra”
```

### Deleting Properties

There is only only way to delete a property, which is to use the *delete* keyword.
Setting it to null or undefined will not remove it.

```javascript
delete myVarL;
```

### hasOwnProperty()

This method can be used to check if a property belongs to a given object. It may return false when an object's constructor does have the property; it only tells whether the object itself has the property.

### The Prototype Property

All instances of a constructor contain this property. The properties of the 'prototype' property itself are inherited by instances of the constructor.
Object.prototype is at the top of the inheritance chain, so all objects inherit from it. It is a very bad idea to change Object.prototype's properties.

### The 'in' Keyword

Checks for a property in an object. Similar to hasOwnProperty(), but it *does* look up the inheritance chain.

```javascript
var myVar = ['hello', 'world'];
console.log('toString' in myVar);
=> true
```

## Inheritance and Looping

If you run a for/in loop on the properties of an object, it will return inherited properties unless you use a guard clause checking each property with hasOwnProperty().

## Host Objects

These objects are provided by the JavaScript environment. Examples include 'window' and 'document' in the browser.

## The 'this' Keyword

The 'this' keyword is available within the scope of a function, but it refers to the object which called the current function. It does not refer to the function itself, but its containing object.

'this' acts like any other variable, only it can't be modified.

In EcmaScript 3, calling 'this' from a nested function referred to the Head Object. This was a serious mistake that was fixed in ES5. As a workaround, programmers used a variable 'that' to keep track of the value of 'this' inside of nested functions.

'this' can be overridden with call() and apply(), which redefine 'this' to be their first parameter (the object they are targetting).

When using a constructor to create an instance, 'this' refers to the object-to-be (the instance that has not yet been created). This pattern works for both properties and methods.

```javascript
var Person = function(name) {
  this.name = name;
}
```

## Scope

There are three kinds of scope in JavaScript:

1. Global Scope
2. Function Scope
3. Eval Scope

There is no scope created by 'for' or 'while' loops, nor by 'if'.

If a variable is declared without the keyword 'var', it is created in the global scope. This is to be avoided.

```javascript
myVar = "hello"; // this is global
var myVar2 = "world"; // this is local
```

### Lexical Scoping

Objects can look up the inheritance chain to find properties, functions, and methods, but they cannot look into their childrens' scopes. When looking for a value up the inheritance chain, the first found value is returned. Any values in higher scopes are 'shadowed', or 'masked' by the value found in the lowest scope.

Priority of inheritance is defined in the definition (the code), not the state of objects at the time of invocation.

## Prototype

Prototype is a property found in instances created by the Function() constructor. These functions can be used as constructors themselves.

Why should you care?

1. Prototype is what the nine native constructors use to pass methods to their object children.
2. You can use prototypes to orchestrate inheritance.
3. You may inherit code that uses it.
4. It's efficient in that all instances of an object can share a method defined in only one place: the prototype object.

The prototype property is basically a container (an object) for properties and methods you want instances of a given object to inherit. They will inherit the current value of the property (it is updated dynamically).

The values passed into prototype must be complex (not JavaScript primitives).

Instances of an object access the prototype's host object with their 'constructor' property.

```javascript
var myArray = new Array();
Array.prototype.foo = 'bar';

myArray.constructor.prototype.foo;
=> 'bar'

// This is the same as line 4. The 'foo' property is found by inheritance.
myArray.foo;

```

All objects have a common highest ancestor: Object.prototype. Overriding the 'prototype' property on Object will break its functionality.

<br/>

<hr />

<br/>

# The Parts of JavaScript

## The Head/Global Object

This object represents the highest scope in a given JavaScript environment. In the browser, it is 'window'. It contains ALL objects in the environment.

Global properties and variables are contained in the Head Object, and are accessible from any other scope through inheritance.

The Head Object can be referenced by referring to 'window', or using the keyword 'this' from the global scope. Thus, its scope is implied from outside of a function.

```javascript
// These mean the same thing:
alert();
window.alert();
```

### Global Functions

These are provided by the environment.

- decodeURI()
- decodeURIComponent()
- encodeURI()
- encodeURIComponent()
- eval()
- isFinite()
- isNaN()
- parseFloat()
- parseInt()



## Object()

Gives an empty object container. Creating objects with a literal is the preferred syntax.

### Parameters

Object() takes a parameter that is a value. It is null/undefined by default.

### Properties and Methods

Note: ALL JavaScript objects inherit the following properties and methods from Object.prototype.

When defining property keys, one can use dot notation or bracket notation. Bracket notation is more flexible, because it allows for the use of concatenation and reserved words.

#### Properties

- prototype

#### Instance Properties

- constructor

#### Instance Methods

- hasOwnProperty()
- isPrototypeOf()
- propertyIsEnumerable()
- toLocaleString()
- toString()
- valueOf()

<br />

## Function()

Functions always return a value. This will be 'undefined' if another return value is not specified.

Functions define a new scope. Variables within it cannot be seen from objects containing the function, but they *can* be seen by objects it contains. This is called 'lexical scoping'.

There are three ways to define a new function:

1. With the constructor: var myFunction = new Function(){};
2. With a literal, as an expression: var myFunction = function(){};
3. With a literal, as a statement: function myFunction(){};

Functions are 'first class citizens' in JavaScript. They are objects, they can be stored, they have properties, and they can be passed into other functions (higher-order functions).

### Invoking Functions

Functions are invoked with (). This can be done in three ways:

1. As a function - func();
2. As a method - when attached to an object - myVar.func();
3. Using apply() or call()
4. As a constructor:

```javascript
var Cody = function(){};
var cody = new Cody();
console.log(cody);
```

### Anonymous Functions

Functions not given an identifier are anonymous.

```javascript
function(){};
```

### Self-Invoking Functions

Functions that call themselves immediately upon definition. This doesn't work with the Function() constructor.

```javascript
var myFunction = function(input) {
  console.log(input)
}();
```

### Self-Invoking Anonymous Functions

Functions with no identifier that immediately call themselves. These often wrap code to keep it from polluting the global object. Note that it must be wrapped in additional parentheses.

```javascript
(function(){
  // code goes here
})();
```

### Hoisting

A function can be invoked before it is defined, because JavaScript collects the functions before runtime. This is called hoisting. It does not work with function expressions.

### Recursion

Functions may call themselves while executing. This is called recursion.

### Parameters

When creating a function with the constructor, the last param passed in becomes the inside of the function.

```javascript
var myFunction = new Function('foo', 'bar', 'return foo+bar');
```

You can create a function with too few parameters. The missing ones will be treated as 'undefined.'

You can also create a function with too many parameters. The extras will be ignored, although they are accessible from the 'arguments' property.

### Properties and Methods

#### Properties

- prototype

#### Instance Properties

- constructor
- length
- arguments

Arguments has a property 'callee' that contains a reference to the function currently executing. This can be used in creating recursive functions.

Arguments also has a property 'length', containing the number of arguments at invocation. This property is being depricated in favor of arguments.callee.length.

<br />

#### Instance Methods

- apply()
- call()
- toString()

<br />

## Array()

A special type of object with a couple of extra functions.

### Parameters

Any comma-separated parameters passed in will become elements in the array.
If a single integer is passed alone, it will define the length of the array instead of an element value. If you define a high index, the elements before it will be filled in with 'undefined'. Setting an array's length can add or remove values automatically.

### Properties and Methods

#### Properties

- prototype

#### Instance Properties

- constructor
- index
- input
- length

#### Instance Methods

- pop()
- push()
- reverse()
- shift()
- sort()
- splice()
- unshift()
- concat()
- join()
- slice()

<br/>



## String()

### Creating Strings

This will create a string object.
It is best to avoid this syntax due to potential problems with typeof.

```javascript
var myObj = new String('foo') // constructor function
```

These two create a primitive:

```javascript
var myString = String('foo') // without new keyword
var myString = 'foo' // literal
```

### Parameters

Takes one parameter: the string value being created.

### Properties and Methods

#### Properties
- prototype

#### Methods
- fromCharChode()

#### Instance Properties
- constructor
- length

#### Instance Methods
- charAt()
- charCodeAt()
- concat()
- indexOf()
- lastIndexOf()
- localeCompare()
- match()
- quote()
- replace()
- search()
- slice()
- split()
- substr()
- substring()
- toLocaleLowerCase()
- toLocaleUpperCase()
- toLowerCase()
- toUpperCase()
- valueOf()

<br />



## Number()

### Creating Numbers

This will create a number object.
It is best to avoid this syntax due to potential problems with typeof.

```javascript
var myObj = new Number(3) // constructor function
```

These two create a primitive:

```javascript
var myNumber = Number(3); // without new keyword
var myNumber = 3; // literal
```

### Parameters

Takes one parameter: the number value being created.

### Properties and Methods

#### Properties
- MAX_VALUE
- MIN_VALUE
- NaN
- NEGATIVE_INFINITY
- POSITIVE_INFINITY
- prototype

#### Instance Properties
- constructor

#### Instance Methods
- toExponential()
- toFixed()
- toLocaleString()
- toPrecision()
- toString()
- valueOf()



<br />

## Boolean()

### Creating Booleans

This will create a boolean object.
It is best to avoid this syntax due to potential problems with typeof.

```javascript
var myObj = new Boolean(true) // constructor function
```

```javascript
These two create a primitive:
var myBoolean = Boolean(true); // without new keyword
var myBoolean = false; // literal
```

### Parameters

Takes one parameter to be converted into a boolean.

The only false values are: 0, -0, null, false, NaN, undefined, and "".

### Properties and Methods

#### Properties
- prototype

#### Instance Properties
- constructor

#### Instance Methods
- toSource()
- toString()
- valueOf()

#### Notes

If you create a false boolean object with a constructor, it creates an object. Objects convert to true. Thus do not use the constructor with false booleans.



<br />

## Primitives: String, Number, and Boolean

A primitive is an irreducible value.
The Primitives are: number, string, boolean, null, and undefined.

Primitives are stored in memory with their value:

```javascript
var myVarA = 12;
var myVarB = myVarA;
myVarA = 13;

myVarA === 13;
=> true
myVarB === 12;
=> true
```

It is best to write these using the literal form. Using a constructor will result in a typeof result of "object" instead of the actual type.

### Object Wrappers

When you call a property or method on a primitive, it is wrapped in a temporary object.
The temporary object is discarded once the properties have been accessed.



<br />

## Null

Use null to indicate that an object property doesn't have a value.
It means that a value is expected but not yet available.
Calling typeof on it will return 'object'. It is best to test for null with ===.



<br />

## Undefined

Used in two ways:

- Indicate that a variable has no assigned value.
- Indicate that an object property you want to access is not defined or named, and can't be found in the prototype lookup chain.

It is best to let JavaScript itself use undefined. Don't use it yourself.



<br />

## Math

This is a built-in, static object. It is not a constructor and cannot be instantiated.

### Properties and Methods

#### Properties
- E
- LN2
- LN10
- LOG2E
- LOG10E
- PI
- SQRT1_2
- SQRT2

#### Methods
- abs()
- acos()
- asin()
- atan()
- atan2()
- ceil()
- cos()
- exp()
- floor()
- log()
- max()
- min()
- pow()
- random()
- sin()
- sort()
- tan()
