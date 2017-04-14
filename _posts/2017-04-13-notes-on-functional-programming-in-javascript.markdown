---
layout: post
title:  "Notes on functional programming in JavaScript"
date:   2017-04-13 22:17:52 -0400
permalink: /notes-on-functional-programming-in-javascript
categories: fp functional programming javascript
---
What follows are my notes as I've been reading [Prfessor Frisby's Mostly Adequate Guide to Functional Programming](https://github.com/MostlyAdequate/mostly-adequate-guide)

## Some useful vocabulary
**Arity**: _For a function, the number of arguments or parameters it will accept_

  The Arity of a JavaScript function can be determined via `Function.length`
  For example 
  ```javascript
    function(a, b) {}.length // 2
  ```
**Variadic**: _a function that will accept a variable number of arguments_

## Currying and Partials
A **curried** function is a function that will return a new _partially_ applied function until its arity has been satsified.
For example
```javascript
const _ = require('lodash');
const map = _.curry(_.map); // map is a curried a function
const upperCase = map(_, str => str.toUpperCase()); // upperCase is a partially applied map function 
upperCase(['first', 'second', 'last']) // ['FIRST', 'SECOND', 'LAST']
```
Note the usefulness of currying.  New and helpful functions can be created using curried functions in a declaritive and less verbose way.

A **partially** applied function is a function where _some_ of its arguments are bound, and when called with the last of its arguments, the function will execute.  Note the difference with a curried function: A curried function has _no_ arguments bound to it, but will return a partially applied function if called without having its arity satisifed. That is to say, if you call the curried function with fewer than the expected number of arguments.  Curried functions are used to create partially applied functions.  Partially applied functions can then have specific use cases of their curried function: A specific type of iterator, like a map that uppercases all the items; a reduce function that removes all null or undefined values, i.e., a compact function; etc. 

Also note that a partially applied function could also be a curried function.  Indeed with `lodash/fp`, the functional variant of lodash, all utilites are curried by default, and partially applied functions returned from curried functions are themselves curried.  

## Function composition
Function composition is the practice of taking the output of what function and _piping_ it as the input to another function.
One unwieldy way of accomplishing this is to simply pass a function call into the args of another function
```javascript
const f = (a) => ....;
const g = (b) => ....;
const result = f(g('some arg'));
```
This approach however is not so readable and, more importantly, not very portable.  We have to call the functions at once to get the result.  What would be more useful is having a way to create a brand new function that is the composition of two or more different functions. Here's a simple implementations that has an arity of *2*:
```javascript
const compose = (fn, gn) => (
  return (arg) => fn(gn(arg))
);

const fun1 = x => x.toUpperCase();
const fun2 = x => x + '!!!!!';

const scream = compose(fun2, fun1);
srceam('ice cream yay') // "ICE CREAM YAY!!!!!"
```
Libraries like rambda and lodash have _variadic_ versions of compose which can be useful for constructing highly declaritive functions from more basic ones in a minimally verbose way. 


```javascript
// map's composition law
var law = compose(map(f), map(g)) === map(compose(f, g));
```
The above states that composing a partially applied map of `g` and a partially applied map of `f` is the equivalent of composing `g` and `f` (i.e., piping output from `g` into the input of `f`) and then partially applying that composed function to map.


## Hindley-Milner type description
Hindley-Milner is a system for describing the type input and output of a function.  Note that although plain-jane JavaScript is a dynamic language, that does not stop one from using comments to describe the type expectations of a function.  Furthermore, with TypeScript and Flow, a typed variation of JavaScript can be written.  
```
// Fun1 :: String -> String
// Fun2 :: Number -> Number
// Fun3 :: [String] -> [Number]
// Fun4 :: String -> Number
```
This should look fairly straightforward:  Accept arguments of the type given on the left, and produce output of type on the right.

Only slightly more complicated:
```
// gun1 :: String -> String -> Number
// gun2 ::  [a] -> a
```
Note the first one with three different components: Although initially perhaps a bit confusing, all this says is that a 'Number' is eventually returned after two `String` arguments are passed in.  The second one makes no specific claim on the type of, but rather states that an `Array` is expected with items of _any_ type and that an element from that array is returned.  The helpfulness of Hindley-Milner should start to become apparant at this point: They almost describe what the function will do.  For example by inspecting a type signature like `[a] -> a`, we can already guess that perhaps the function will return the first (or last) element in the array; Or perhaps it will return a element that meets some predefined criteria.  

It should be noted that like function composition, the associative property holds for Hindley-Milner type signatures.  Parenthesis can help describe intent.

Some examples from https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html#tales-from-the-cryptic

```javascript
//  strLength :: String -> Number
var strLength = function(s) {
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs) {
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
var match = curry(function(reg, s) {
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s) {
  return s.replace(reg, sub);
});

//  match :: Regex -> (String -> [String])
var match = curry(function(reg, s) {
  return s.match(reg);
});


//  onHoliday :: String -> [String]
var onHoliday = match(/holiday/ig);

//  replace :: Regex -> (String -> (String -> String))
var replace = curry(function(reg, sub, s) {
  return s.replace(reg, sub);
});

//  id :: a -> a
var id = function(x) {
  return x;
}

//  map :: (a -> b) -> [a] -> [b]
var map = curry(function(f, xs) {
  return xs.map(f);
});

//  head :: [a] -> a
var head = function(xs) {
  return xs[0];
};

//  filter :: (a -> Bool) -> [a] -> [a]
var filter = curry(function(f, xs) {
  return xs.filter(f);
});

//  reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs) {
  return xs.reduce(f, x);
});
```
