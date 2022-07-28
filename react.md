# React

Way to build stateful, reusable, fast-rendered UI components.

 * Declarative
 * Flexible
 * Efficient/Virtual DOM:
   * Can be rendered both on the server and the client side
   * Minimize updates to real DOM

# JSX

Preprocessor that adds XML syntax to javascript.
JSX tags can have
  * a name
  * properties
  * children


# Components

The heart and soul of react.
Created with ```React.createClass```.
Must implement a "render()" function that returns an html string.

## Props
Jsx properties can be accessed inside a component via ```this.props```.

Components are rendered into the browser via the method ReactDOM.render(component, element);

## Events

Just add onXYZ to elements in component. Handlers are defined inline inside the component.

```
var BannerAd = React.createClass({
  onBannerClick: function(evt) {
    // codez to make the moneys
  },

  render: function() {
    // Render the div with an onClick prop (value is a function)
    return <div onClick={this.onBannerClick}>Click Me!</div>;
  }
});
```

## State

State is internal data handled by the component itself (as opposed to props
which are passed in from parent component)


```getInitialState```: implement to return inital state
```this.state```: call to get current state
```this.setState```: call to alter state

If the state changes, the "render" function will be called with the new state.

# Virtual DOM
DOM manipulation is slow. React components updates a virtual DOM instead of the real DOM.
When all components have updated, a diff is performend and the actual changes are
applicated on the real DOM in a single step.





