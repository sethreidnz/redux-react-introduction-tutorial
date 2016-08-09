# What is Redux and why do I need it?

In the past web sites were simple server side rendered post-back based user interaction. As JavaScript evolved
and 'web apps' started to content with the native mobile experience we had an explosion of frameworks and tools
such as Backbone, Knockout, Durandal, Angular, Ember and so on. 

When React came along many of the other frameworks took heed of what was done in terms of uni-directional data flow
and a more functional approach. This solved a lot of problems but one thing that is still hard in any non-trivial
application is managing the data and the UI state and passing data around between interdependent components.

This is where Flux and Redux come it.

## Flux

[Flux](https://facebook.github.io/flux) is used internally by Facebook for building client side apps. As the docs say:

> It's more of a pattern rather than a formal framework, and you can start using Flux immediately without a lot of new code.

One doesn't need to understand Flux to use Redux but since it is heavily based on Flux it is worth a brief explanation.

The Flux architecture eschews MVC has three main parts:

- Views
- Stores
- The dispatcher

![Flux uni-directional flow](images/plux-uni-directional-flow.png)

As you can see in the image the action goes through the dispatcher, to the store and data flows into the view. This uni-directional model
is the core tenant of Flux.

### Views

As described in [the first part of this tutorial](https://github.com/justsayno/react-introduction-tutorial) the recommended pattern
for building React apps is creating a distinction between `Presentational Components` and `Container Components`. These
make up the "views" part of our Flux application.

### Stores

This is where data is stored and flows *from* the store into the views. The views cannot directory update the state, instead they
do this through dispatching an action. The store registered callbacks with the dispatcher which are functions that take an action type.
Inside this function there is a switch statement determining what will happen to the store's state as a result of the action.

### Dispatcher

As described when an event or a user interaction occurs the component will emmit an `Action`, the event is 'dispatched' and any callbacks registered
by the `Stores` are run with the dispatched action as a parameter. Once the store's callback(s) have been run then the new state will flow into the views.

### What if a component wants to update the state?

The only way that a view can update the state is via and action. The store does not have any direct set methods and the only thing that
can act on the state are callbacks registered with the dispatcher.

![Flux uni-directional flow](images/plux-uni-directional-flow-2.png)

### Action creators

Generally it is common to wrap up the logic for creating an action into a function so that any component that needs to create that 
action can have this function passed to it as props.

``` javascript

export MyComponent = ({onClick}) => (
    <a onClick={onClick} className="btn">Click me</a>
)

MyComponent.propTypes = {
    onClick = PropTypes.func
}

```

In the above example the prop `onClick` would be a Flux action creator function. This allows complete de-coupling of the 
logic around how state is mutated and the interactions that initiate it. This example is also an example of a Presentational
component that contains no logic of its own and has its action creators and other props passed to it from its parent component.

That is enough background on Flux. Lets talk about Redux in the next step.

Next Step - [Click Here]()