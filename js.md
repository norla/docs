## JS has function scope

Each function is it's own scope.
If you need smaller scope, for example in a for loop: Use IIFE:s.

## JS is compiled
Two step-process to execut js code.

Compilation
  * looks for formal declarations and put identifiers in their correct scope.
  * generates bytecode
  
Compilation must be super fast (microseconds)

.js file -> [compiler] -> bytecode -> [virtual machine]

Both compiler and vm talk to the scope manager to define/resolve identifiers.

## Formal declarations:

  * An expression where the first word literally is "function" or "var"
  * Function parameters.

This is basically is the js compiler is interested in.

Function expressions:

  * When the word "function" is not first in the expression.
  * Named are prefered to unnamed:
     * Function can then be accesed from itself for recursion.
     * Function name shows up in stack traces.

```
var foo = function bar() {
  // bar is a read only and SAFE reference now 
  // foo is a global UNSAFE reference
}

// bar is not visible here
```

## Types of scope

### Lexical scope
Static, compile-time scope.
Author determines scope of variables.
All lookups can be cached for better performance since everything is static.

Used in JS EXCEPT:
  * eval("var a = 10;") // Lexical scope is modified in runtime!
  * with (obj)
  
If you use any of these all lexical scope optimizations are gone.
  

### Dynamic scope 
Run-time scope. Not commonly used. Hard to optimize.
Finds outer scopes by going up the call stack (as oppposed to looking sourrounding code).

## Block scope
let, const

Const does NOT enable immutable values. Just variables that cant be redeclared.

## undefined, undeclared

undeclared: not found in any formal declarations
undefined: it has been declared but has not been given a value

## Hoisting

Variable declarations are magically moved to top of scope;
Side-effect of two-step compile/run process in js. 

Bad for vars

```
a=2;
...
var a;
```

Good for functions
```
foo();
...

function foo() {

}

```

## Closure

When a _function_ remembers its lexical scope even when it's execute it outside that scope.
If two functions close over the same variable, they both reference the exact same variable.

### Module pattern

An outer function that runs at least once and returns at least one inner function that closes
over it's internal state.

```
var module = (function() {
   var state = {name: "Hej"};
   return {
      bar: function() {
         return state.name;
      }
   })();
   
console.log(module.bar());

```


## this

What this points at is determined by HOW the function is called.

You cannot answer what "this" means simply by looking at the function. 
You have to look at the call site to do this. Sorta like dynamic scope.

### Why?

More flexible than lexical binding.
A function can be shared across multiple contexts easily.

### How?

There a 4 rules to determine this

* 1. New keyword (see below)
* 2. Explicit rule: use function.call(...) to set this explicitly
     Hard binding: using object.bind() to give a function a hard binding before passing it along
* 3. Implicit rule: If there is a owning object at the call site: use the owning object as "this".
* 4. Default rule: if in strict mode: global object, otherwise: "global object"

```js
function foo() {
  console.log(this.bar);
}

var bar1 = "bar1",
var o2 = {bar: "bar2", foo: foo};
var o2 = {bar: "bar2", foo: foo};
  
foo();     // rule 4 -> "bar1"
o2.foo();  // rule 3 -> "bar2"
o2.foo();  // rule 3 -> "bar3"

foo.call({bar: "bar4"}); // rule 2 -> "bar4"

baz(foo.bar); // rule4 (ctx is lost!) -> "bar1"
baz(foo.bar.bind({bar: "bar5"})); // rule 2 -> "bar5" 

function baz(cb) {
  cb();
}
```


### new

Does 4 things when put in front of a function call

* Creates a new empty object out of thin air.
* It links that object to another object. See: "prototypes" below
* The function is called with that created object as "this"
* When the function is finished: if the function did not return an object,
  the "new" keyword assumes you wanted to return the new object.
  
## Proptotypes

### Class theory

Class     - object
Blueprint - building

The building has been copied from the blueprint, but is then a separate entity.
This is called instantiation.
Removing a window from the building does not remove it from the blueprint.

- - -

prototype - object

Is however not the same thnig. We are not copying down, but referencing up.
Aka if you remove a window from the building, it disappears from the blueprint.

### Prototype

Term used for three different things:

#### Function.prototype

Each function has one. Linked object created by "new".

#### [[Prototype]]

Term for link to next object in protoype chain. Created by "new", or "Object.create".

#### __proto__

Property on object for next object in chain.

### Super unicorn magic:

The "this" keyword maintains it's binding when you walk up the prototype chain.

#### Object.create(obj)

1. Creates a new object
2. Sets the supplied parameter as prototype for the new object.

  









