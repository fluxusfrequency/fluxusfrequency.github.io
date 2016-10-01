---
title: CoffeeScript Notes
date: 2013-11-03 08:31:00 Z
categories:
- source
layout: post
comments: true
---

I've been digging into a lot of JavaScript lately, so I thought it would be worthwhile to read [The Little Book on CoffeeScript](http:# arcturo.github.io/library/coffeescript/) by Alex MacCaw.

I took some notes as I went, and I thought I'd share them here for easy reference.

<br />


##Basic Syntax

```coffeescript CoffeeScript - Basic Syntax
# Functions
myFunction = ->
# Translates to:
function() {



# Functions with arguments
myFunction = (arg) ->
# Translates to:
function(arg) {



# No semicolons are needed in CoffeeScript(CS).



# Maintaining the context of 'this' with the 'fat arrow'
myFunction = (arg) =>



# Operators

is
# Translates to:
\===

not
# Translates to:
!

isnt
# Translates to:
!==

and
# Translates to:
&&

or
# Translates to:
||



# Single-line "if" statements
if x then y



# Splat operator
(…)



# For loops
myFunction(item) for item in array



# While loops
while num -= 1



# Alias for prototype
Bob::hey
# Translates to:
Bob.prototype.hey



# Executing functions immediately with the keyword do
type = do ->
# Code is then private and cannot be accessed in the future.



# Chained comparison syntax
a < b < c



# Null Checks
blackKnight.getLegs()?.kick()
# Won’t call kick() if getLegs is false.
blackKnight.getLegs().kick?()
# Won’t call kick() if kick() is null or not a function.
```

##Ruby-Like Features

```coffeescript CoffeeScript - Ruby-Like Features
# Declaring a variable
string = "Foo bar baz"



# Inline "if" statments
console.log("Hello, world!") if true



# "unless" statments
console.log("Hello, world!") unless false



# “==“ converts to “===“, “!=“ converts to “!==“
1 == 2
# Evaluates as "1 === 2"
1 != 2
# Evaluates as "1 !== 2"



# String Interpolation
code = "Hello"
console.log(“#{code}, world!”)



# Defining an object literal without curly brackets
obj = one: 1, two: 2



# Conditional assignment operator
variable ||= 2
# Is the same as:
variable or=2
# You can also use:
variable ?= 2



# Instance variable
@snake
# Evaluates to:
this.snake



# Ranges
[1..5]



# Slice
[1, 2, 3, 4, 5][0..1] => [1, 2]
# Or:
“my string”[1..4]



# Checking Content
arr [“a”, “b”, “c”]; “a” in arr
# Returns true.
# Like Ruby #include?



# Existential Operator
praise if brian?
#  Only returns false for null or undefined (*NOT* for “” or 0).
#  Like .nil? in Ruby.
#  Can also be used for ||.
```

##Classes

```coffeescript CoffeeScript - Classes
# Declaring Classes
class Animal



# Instantiating Classes
animal = new Animal(args)



# Instantiating Classes With Instance Variables
constructor: (arg) ->
@name = name
# Or the shorthand:
contructor: (@arg) ->



# Inheritance
class Parrot extends Animal
# In Ruby, this would be: class Parrot < Animal
```

##Enumerables

```coffeescript CoffeeScript - Enumerables
# each
myFunction(item) for item in array



# each with index
for name, i in names



# map
result = (item.name for item in array)



# select
result = (item.name for item in array when item.name == “test”)



# includes
included = “a long test string”.indexOf(“test”) isn’t -1



# iterating
alert(“#{key}#{value}) for key, value of object



# min/max
Math.max[1,15, 2312, 42]… => 2312
```

##Bad Parts

```coffeescript CoffeeScript - Bad Parts
eval()
# Executes a piece of Javascript code in local scope. Throws complier off.
# Insecure: this opens code for injection attacks.
# Instead of:
model= eval(modelName)
# Use:
model = window[modelName]



typeof()
# Only useful to check if something is undefined, otherwise results are inconsistent.
# Instead of:
typeof(obj)
# Use:
Object.prototype.toString()
# Use duck typing with existential operator to try using an object whose type you don’t know.



instanceof()
# Broken, like typeof(). Avoid use.



delete
# Can be used safely to remove properties within objects, but not to delete functions or variables.
# Assign them to null instead.



parseInt()
# Pass it a base to ensure it works as expected.
parseInt(‘num’, 10)
```


##Good Parts

- Semicolons - don’t need them in CoffeeScript!
- Global variables - CS automatically declares var to prevent them!
- Reserved words - CS automatically escapes them!
- Type coercion - CS turns "==" into "==="
- Declarative functions - although you can call a function before it’s defined in JS, CS removes this problematic behavior.
- Number property lookups - in JS, "." gets interpreted as a decimal. In CS it gets changed to "..", to access a property.
- Linting - CS has built in linting, which can be called with
"coffee lint index.coffee"

