# Redux

> There is great documentation on the [Redux site](http://redux.js.org/docs/introduction/index.html) and 
> I highly recommend watching Redux author Dan Abramov's [Getting Started with Redux](https://egghead.io/courses/getting-started-with-redux) 
> and [Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux).

Redux is a state management library that is very simple and has three main principals. It is so simple the author
[Dan Abramov]() made a [90 line version](https://gist.github.com/gaearon/ffd88b0e4f00b22c3159) which is the bare
bones of the library (sans some edge case handling).

It is made up of a number of concepts that at first can seem daunting but are actually quite simple. In short
when using Redux you store all your application state in one JavaScript object (the store). This state is read-only
and the only way to change it is to emit an `Action` via the `store.dispatch()` function. When an action is dispatches the current state and the action are passed
to what is called a 'reducer'. A reducer is a function that takes the current state and the action that has been dispatched and returns
the new state. 

The reducer does not mutate the state but instead creates a new state with the changes required. These changes then flow down to your
components.

## Three principals of Redux

There are three basic principals of Redux.

### 1 Single source of truth

> The state of your entire application is stored in an object tree with a single [Redux store](http://redux.js.org/docs/Glossary.html#store)

This makes it really easy to make universal apps that can render on the server as well as making it easier to debug
since every action in your application can be tracked and stepped through after the fact. For example this could be
the shape of my application state:

```
{
    employees:[]
    isFetching: false,
    hasLoaded: false
}
```

I have an array `employees` to store my employee data, boolean `isFetching` to store the status of fetching the employee
data and `hasLoaded` as a flag to indicate when all activity relating to loading the employee data for display is complete.

### 2 State is read-only

The 'single source of truth' state is read only. This means that in order to change the state of the application
you have to create a new version of the state object with the changes required. This can be done in various ways
including using [immutable.js](https://facebook.github.io/immutable-js/) or with 
ES2015 [Object.assign()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign). 
For simplicities sake in this tutorial we are going to use `Object.assign()`. 

##### Actions

The only way that state can be mutated is through using the redux `store.dispatch()` function to 
create an action. An action is just a plain JavaScript object that has a `type` property that describes the action. For
example imagine that the application has loaded and the employee data is busy being fetched so the state looks like this:

```
{
    employees:[]
    isFetching: true,
    hasLoaded: false
}
```

When the data is received we would emit an Action like this:

```
store.dispatch({
  type: 'REQUEST_EMPLOYEES',
  payload: [{...}]
})
```

Where payload could be any key on the state object and I've omitted the detail on the employee data.
This would get passed to a reducer functio.

##### Reducers

[Reducers](http://redux.js.org/docs/Glossary.html#reducer) are a functions that take the previous state as the first argument 
and the object that describes the action that has been dispatched as the second.

``` javascript
const reducer = (state, action) => {
    return state
}
```

The above example is the simplest reducer where we just return the previous state. Now we need to add the logic to 
get the `payload` of employees into our current state.

### 3rd Principle -  Changes are made with pure functions

Pure functions are functions that do not have any observable side effects such as database calls, network calls or
shared session data. In our example we could do this:

``` javascript
const reducer = (state, action) => {
    state.employees = action.payload;
    state.isFetching = false;
    state.hasLoaded = true;
    return state;
}
```

But this would change the original state object and break the rule of it being a pure function. This principal is nesisarry
to get some of the advantages of Redux such as the [Redux Dev Tools]() I will show you later.

The better way to do this is to use Object.assing. 

> In larger more serious apps its better to use [immutable.js](https://facebook.github.io/immutable-js/) to boost performance and memory usable.

``` javascript
const reducer = (state, action) => {
    return Object.assign({}, state, {
        employees: action.payload,
        isFetching: false,
        hasLoaded: true
    });
}
```

Now the state will look like:

``` javascript
{
    employees: {...},
    isFetching: false,
    hasLoaded: true
}
```

So that is the three principals:

1. The entire state of your application is stored in on state object
2. The state is read-only. The only way to change it is to emit an action
3. Changes are made with pure functions.

Lets get into building this idea into our app. I have only scratched the surface here but I think it is easier
to understand Redux by example so lets get started.

Next step - [Click here]()