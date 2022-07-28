No help for async in react.

Rxjs can make it managable. Not beatiful.

Redux 
-----
predicable state management using actions and reducers.

Action: WHAT you want to do. jsut the intent.
Reducres: pure function (state, action) -> newState

Examples of hard things:
o Cancelling AJAX calls.
o Throttling of clicks.


Promises
--------
Uncanceble.
Single return value.


Observalbles
------------
Stream of zero, one ore more values. Over time.
Cancellable.

RxJS is a reference implentation of observables.

Instead of "then" -> subscribe(errCallback, valueCallback, doneCallback)

Trivial to throttle, buffer, cancel etc.

Docs 
http://reactivex.io/documentation/operators.html
https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md
https://gist.github.com/staltz/868e7e9bc2a7b8c1f754

Operators on observables
------------------------

Operators usually take an observable and return a modified observable.
Operators are chained and applied in turn one after the other. 

Creating new observalbles:

* fromXXX - convert an object of type x into an observable
* just - create single observable form argument
* of - create observable sequence from argument list

Combining:

* merge: combine multiple observables into one by merging their emissions as they are emitted.
* concat: similiar to merge but emits all events from the first observable first,
          then all events from the second etc.

Transforming (instance methods in rxjs):

map - one-to-one
filter - many-to-fewer
mergeMap/flatMap - many-to-more

* buffer - Gather items from observable and emit a bundle instead of single items.
* map - Applies a function to each item emitted, return observable that emits the result.
        Provided function is "observable => plain value"
        Use when you only need to emit static, plain values.
* flatMap - Transform items emitted by observable into new observables.
            flattens these emissions from these new observables into one.
            Provided function is "observable => observable"
            Use when you create several new observables using merge(), of() or fromXXX() etc
            Same as calling map(fn).mergeAll()

Filtering (instance methods in rxjs):

* filter: emit only items matching predicate test
* debounce: throttle frequency of item

Error handling:

* catch: recover from onError notification and continue sequence.


RxJS + Redux
------------

Epic: function "stream of  all dispatched actions" -> "stream of new actions to dispatch".

Epics are run after the actions are dispatched to the reducer, so they cannot be "swallowed".

const someEpic(action$, store) {

// action$ is a stream of action objects
// we should now modify it to make it return another stream of action objects

  
}


