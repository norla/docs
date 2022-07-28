## One store

One big javascript object with several properties.
Never mutates, only create new versions of it.

Object with 

 * property named "state"    (global read-only state) 
 * function named "dispatch" (for dispatching events)

## Components

### Provider component

Re-renders entire app when store changes.

### Containers - Smart components

Aware of redux. Called "containers".


### Dumb components

Gets data as props. Redux-unaware.

## Actions

Payloads of information that send data from application to the store.
Are usually created by calling an "action creator" where also business logic will reside.

## Reducers

Specifies how the state changes in response to actions.
Pure function on the format

```
(previousState, action) => newState
```

Should not contain business logic and just focus on keeping
the global state consistent.

## Middleware

Third party extension point between a dispatched ACTION and the REDUCER.

Examples:

### Thunk
Allows an action function to be dispatched instead of an action object.
The function should return an action object and will be called with the
state and dispatch props of the store as arguments.

## React integration

Redux is integrated into react by finding the top-level components and set their state
to redux's state. They will then be automatically re-rendered when the redux's state changes.

This is done using the ```connect(mapStateToProps, mapDispatchToProps)``` function. These two functions
will add stuff to the components props so that it can use the global state and dispatcher 
however it likes.

The top components can then pass parts of the state down to children as props. Children are then 
100% redux-unaware.

index.js:
```
import { Provider } from 'react-redux';
import App from './containers/App';

const store = configureStore();

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

App.js:

```
import React, { Component, PropTypes } from 'react';
import { connect } from 'react-redux';

class App extends Component {
  render() {
    const { todos, actions } = this.props;
    return (
      <div>
        <DumbComponent1 addTodo={actions.addTodo} />
        <DumbComponent2 todos={todos} actions={actions} />
      </div>
    );
  }
}

App.propTypes = {
  todos: PropTypes.array.isRequired,
  actions: PropTypes.object.isRequired
};

function mapStateToProps(state) {
  return {
    todos: state.todos
  };
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(...some action generators..., dispatch)
  };
}

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(App);

```

### Selectors

If uncomfortable exposing all the state to components, use the selectors pattern instead.

"Reading API" used by components to fetch data from the store. 
Should be co-located with reducers. Minimal amount of data should be
kept in the store. Derived data should be calculated by selectors.

Selectors are composable and can be by combining other selectors.

Example:

```
import * as selectors from "../selectors";

...

function mapStateToProps(state) {
  return { current: selectors.getCurrent(state) }
}
```
