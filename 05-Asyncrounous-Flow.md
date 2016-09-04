# Asyncronous flow

```
git checkout step-5
```

There are a number of solutions to how to do asyncronousy in Redux. Since reducers are pure
functions we can't have and direct side effects as part of our call. It also needs to return 
the next state so we can't exactly return a promise or something like that.

The three main options are:

1. [Redux Thunk](https://github.com/gaearon/redux-thunk)
2. [Redux Sagas](https://github.com/yelouafi/redux-saga)
3. [Redux Loop](https://github.com/redux-loop/redux-loop)

Redux Thunk is probably the simplest to get started with so this tutorial is going to focus on using that.

## Redux Thunk

Redux Thunk is what is called Reudx Middleware. Middleware allows you to enhance the Redux store in some way.
In this case Redux Think allows your action creators to return not just an object representing the action
being created. But you could return a function whos signature looks like this:

``` javascript
//arrow function
(dispatch, getState) => {
 
}
//or
function(dispatch, getState){

}
```

So for example I could convert this:

``` javascript
export const requestEmployees = () => ({ 
    type: EMPLOYEES_REQUESTED
})
```

Into this:

``` javascript
export const requestEmployees = () => {
    return (dispatch, getState) => {
        dispatch({ 
            type: EMPLOYEES_REQUESTED
        })
    }
}
```

The function we return has passed to it two store arguments:

- dispatch
- getState

## Configuring Redux Midddleware

First we need to add the Redux Thunk package using npm and then we need to tell Redux about it. You do this by passing the middelware
to as a second argument to the Redux `createStore` function. 

``` javascript
const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
)
```

However since we are already using the Redux Dev Tools Middleware we need to use another Redux function called `compose`. It's not
really important how this works but it lets us use multiple Redux Middlewares.

``` javascript
export const store = createStore(
  employeeReducer,
  initialState,
  compose(
    applyMiddleware(thunk),
    window.devToolsExtension ? window.devToolsExtension() : f => f
    )
);
```

These let us inspect the current state as well as dispatch actions in our action creator. The above example is not very useful 
but say we turned the `getEmployee` into a function that returns we could call it like this:

``` javascript 
// Actions
const EMPLOYEES_REQUESTED = 'EMPLOYEES_REQUESTED'
const EMPLOYEES_RECEIVED = "EMPLOYEES_RECEIVED"

// Action Creators
export const employeesRequested = () => ({
    type: EMPLOYEES_REQUESTED
})

export const employeesReceived = (employees) => ({
    type: EMPLOYEES_RECEIVED,
    employees: employees
})

export const requestEmployees = () => {
    return (dispatch) => {
        dispatch(employeesRequested())
        return getEmployees().then(
            (employees) => dispatch(employeesReceived(employees))
        )
    };
}
```

Notice how I have created two actions now, one for when the employees are requested and one for when the employee data is recieved
I've also changed the reducer now to handle this:

``` javascript
export const employeeReducer = (state = initialState, action) => {
    switch (action.type) {
        case EMPLOYEES_RECEIVED: {
            return Object.assign({}, state, {
                employees: action.employees
            })
        }
        default: {
            return state
        }
    }
}
```

For the moment I am not handling the case for `EMPLOYEES_REQUESTED` because there isn't much to do there.
But what I can add is a spinner to indicate that it is loading:

```
git checkout step-5-1
```

To do this I need add a hasLoaded flag to my initial state:

``` javascript
const initialState = {
    employees: [],
    hasLoaded: false
}
```

This flag will be `false` to begin with and when the data is recieved then it will
be `true`. So in my reducer in the case `EMPLOYEES_REQUESTED` I will set `hasLoaded` to false,
and in the `EMPLOYEES_REQUESTED_SUCCESS` i set it to true.

``` javascript
export const employeeReducer = (state = initialState, action) => {
    switch (action.type) {
        case EMPLOYEES_REQUESTED: {
            return Object.assign({}, state, {
                hasLoaded: false
            })
        }
        case EMPLOYEES_REQUESTED_SUCCESS: {
            return Object.assign({}, state, {
                employees: action.employees,
                hasLoaded: true
            })
        }
        default: {
            return state
        }
    }
}
```

And I have added a simple CSS spinner by creating a `Spinner` component and adding some CSS to index.css.

``` javascript
import React from 'react'

const EmployeeListItem = () => (
    <div className="sk-cube-grid">
        <div className="sk-cube sk-cube1"></div>
        <div className="sk-cube sk-cube2"></div>
        <div className="sk-cube sk-cube3"></div>
        <div className="sk-cube sk-cube4"></div>
        <div className="sk-cube sk-cube5"></div>
        <div className="sk-cube sk-cube6"></div>
        <div className="sk-cube sk-cube7"></div>
        <div className="sk-cube sk-cube8"></div>
        <div className="sk-cube sk-cube9"></div>
    </div>
)

export default EmployeeListItem
```

I then just mapped the hasLoaded from state to props and conditionally display the spinner if it has not loaded yet:

``` javascript

{..} //excluded for brevity

class EmployeeDashboard extends Component {
  constructor({requestEmployeesAsync}){
    super()
    requestEmployeesAsync()
  }
  render() {
    let { employees, hasLoaded } = this.props
    if(!hasLoaded){
      return <Spinner />
    }
    return (
      <div className="employee-dashboard col s12 m7">
            <EmployeeList>
             {employees.map((employee) => {
                return <EmployeeListItem key={employee.id} employee={employee} />
              })}
            </EmployeeList>
      </div>
    )
  }
}

{..} //excluded for brevity

const mapStateToProps = (state) => ({
  employees: state.employees,
  hasLoaded: state.hasLoaded
})

```


## Sharing state between components

At the moment I am still using the old `getEmployee(employeeId)` call for the single user page but this
won't work any more now that we have turned `getEmployees` into an asyc call. We could do another asyncronous
call to get the single employee but since the data is the same we might as well just share state and 
filter the one we want.

So I have added Redux `connect()` to `EmployeeProfile` alon with `mapStateToProps`

``` javascript
const mapStateToProps = (state) => ({ 
    employees: state.employees
})

export default connect(mapStateToProps, null)(EmployeeProfile)
```

I've also moved the logic that I was using to get the selected employee into its own function inside my EmployeeProfile
class so that it reads more clearly:

``` javascript
_getSelectedEmployee(props){
    // get employee and employee id from props
    const { employees, params: { employeeId } } = props
    
    // filter employees for the one that is selected
    return employees.filter((value) => {
        return value && (value.id === employeeId)
    })[0]
}
```

One problem with this is if you reload the EmployeePofile route then you get an error... This is because
it is not actually retrieving the employees so the filter function fails to return an employee.

We could fix this by simply calling the `requestEmployees` action creator in both the EmployeeDashbaord component
and the EmployeePofile component but then we would loose the caching that Redux is affording us.

In the next step we will look at using Redux Thunk to 'bail out' of an action request.

[Next Step](06-Conditional-Actions.md)