# Principles

## Avoid side effects

A funcition should have direct input only (parameters) and direct output only (return value).
This makes it trustable and readable and mathematically sound.

* We can't be without them. IO is a side effect.
* Side affects should be in the outer shell of the program, keeping them all in the same place.
* The core should be side-effect free.

### Side cause 

If a function relies on any state that we did not pass in as parameters.
This means that we cant understand the function without looking at other 
parts of the program.

### Side effect
Function changes state of the program other than it's return values.

### Pure functions

Three definition

 * No side causes/effects
 * Always return the same output given the same inputs.
 * Refenctial transparancy. Any function call can be replaced with it's output and the 
   program will behave the same way. Once a function is computed we can use the return value
   every time we see it being called with the same args again.
   
In javascript writing 100% pure functions is virtually impossible. We have to live with some
level of uncertainty.
   
## Composition

Combining functions in a declarative way using "function factories".
No intermediate variables used. Only works if all functions take one argument.
If they take multiple args, currying can be used.

```
var x = foo(1);
var y = bar(x);
var z = baz(y);

// becomes ->

var z = baz(bar(foo(1)));

// becomes ->

var z = pipe([foo, bar, baz], 1)

```

## Immutability

### Constant
variable that cannot be reassigned.
immutable variable assignment

```
const f = [1,2,3];
f = [2]; // illegal
f[0] = 4; // legal
```
mutated variable assignment is easy to spot, can only occur in lexical scope.

const is therefore quite limited.

### Immutable values
immutable values

```
var f = Object.freeze([0,1,2]);
f[0] = 4; // illegal

```

Mutated values are super hard to track down.

## Closure

Can be used in an unpure way to create functions that mutate the values it closes over.

## Partial application

```
function apa(a,b,c) {
    ...
}

var f2 = partial(apa, 1, 2));
var result = f2(3);
```

## Currying

Not the same as partial application!

Only creates functions that take ONE argument.

```
var fun = curry(apa) 
var res = fun(1)(2)(3)
```

Makes it simpler to compose functions when they all take one argument.

## Lists

May problems can be modeled as list operations, avoiding imperative code
and intermidiate variables.




