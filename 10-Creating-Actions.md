# Creating an action

```
git checkout step-1
```

Actions in Redux are just simple objects that describe the action with their `type` key. It is common to use
a string as the type for the action:

``` javascript
export const REQEUST_EMPLOYEES = 'REQEUST_EMPLOYEES'
```

I have created a new folder `src/actions` and it it creates a file `employees.js` which contains the following:

``` javascript
// Actions
export const REQEUST_EMPLOYEES = 'REQEUST_EMPLOYEES'

// Action Creators
export const requestEmployees = () => {
  return {
    type: REQEUST_EMPLOYEES
  }
}
```

Here I have defined the contant for the Action and created an action creator `requestEmployees` that consumers could
pass to the `dispatch` function like this:

``` javascript
dispatch(requestEmployees())
```

But we haven't even installed redux yet! Lets do that by using npm to install `redux` and `react-redux`:

```
npm install --save redux react-redux
```

An

