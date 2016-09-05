# What is Redux and why do I need it?

In the past web sites mostly involved simple server side rendered, post-back based, user interaction. As JavaScript evolved
and 'web apps' started to contend with the native mobile experience there has been an explosion of frameworks and tools
such as Backbone, Knockout, Angular, Ember and so on.

These frameworks and libraries solved many problems such as data binding in between the view (html) and the model (data) and how to manage dependencies. 
Often the missing piece is a simple, scalable way of dealing with application state as your project grew and with it the complexity of the state.

The type of state I am talking about involves data from the server, UI interactions, forms and everything else that comes with any non-trivial application.

This is where Facebook came up with the Flux architecture, which later inspired Redux. It draws many parallels to the concepts in [Command Query Responsibility Segregation](https://msdn.microsoft.com/en-us/library/dn568103.aspx)

## Flux

[Flux](https://facebook.github.io/flux) is used internally by Facebook for building client side apps:

> It's more of a pattern rather than a formal framework, and you can start using Flux immediately without a lot of new code.

One doesn't need to understand Flux to use Redux but since it is heavily influenced by Flux it is worth a brief explanation.

The Flux architecture eschews MVC has four main concepts:

- Views
- Stores
- Actions
- The dispatcher

![Flux uni-directional flow](images/plux-uni-directional-flow.png)

As you can see in the image the action goes through the dispatcher, to the store and data flows into the view. This uni-directional model
is the core tenant of Flux.

### Views

As described in [the first part of this tutorial](https://github.com/justsayno/react-introduction-tutorial) the recommended pattern
for building React apps is creating a distinction between `Presentational Components` and `Container Components`. These combined
make up the "views" part of our Flux application.

### Stores

This is where state of the flux application is maintained and it flows **from** the store into the views. The views cannot directory update the state, instead they
do this through dispatching an action. 

The store registers callbacks with the dispatcher which are just functions that take an action type and any other data about the action.
Inside this function there is a switch statement determining what will happen to the store's state as a result of the action.

There can be multiple stores in a Flux application and each store might have a unique callback registered for a particular action. Stores can
 act as a local caching mechanism in the app.

### Dispatcher

When an event or a user interaction occurs the component will emmit an `Action` via the dispatcher and any callbacks registered
by the `Stores` are run with the dispatched action as a parameter. Once the store's callback(s) have been run the new state will flow into the views.
One thing to note is that there is only one dispatcher in an application.

### What if a component wants to update the state?

The only way that a view can update the state is by emitting an action. The store does not have any direct set methods and the only thing that
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

In the above example the prop `onClick` could be a Flux action creator passed to it from its parent. This allows complete de-coupling of the 
logic around how state is mutated and the interactions that initiate it. This example is also an example of a Presentational
component that contains no logic of its own, just props and presentation.

That is enough background on Flux. Lets talk about Redux in the next step.

Next Step - [Click Here]()