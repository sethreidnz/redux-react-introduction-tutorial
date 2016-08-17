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

Redux Thunk is by far the simplest to get started with so this tutorial is going to focus on using that.

## Redux Thunk

Redux Thunk is what is called 'middleware' and it allows you to write action creators that return a 
function instead of an action. So instead of our action creator looking like this:

``` javascript
export const requestEmployees = () => ({ 
    type: REQUEST_EMPLOYEES
})
```

We could do this:

``` javascript
export const requestEmployees = () => {
    return (dispatch) => {
        dispatch({ 
            type: REQUEST_EMPLOYEES
        })
    }
}
```

The above example is not very useful but say we turned the `getEmployee` into a function
that returns we could call it like this:

``` javascript 
// Actions
const REQUEST_EMPLOYEES = 'REQUEST_EMPLOYEES'
const REQUEST_EMPLOYEES_SUCCESS = "REQUEST_EMPLOYEES_SUCCESS"

// Action Creators
export const requestEmployees = () => ({
    type: REQUEST_EMPLOYEES
})

export const requestEmployeesSuccess = (employees) => ({
    type: REQUEST_EMPLOYEES_SUCCESS,
    employees: employees
})

export const requestEmployeesAsync = () => {
    return (dispatch) => {
        dispatch(requestEmployees())
        return getEmployees().then(
            (employees) => dispatch(requestEmployeesSuccess(employees))
        )
    };
}
```

Notice how I have created two actions now, one for requesting and one for the successful
retrieval of the data. I've also changed the reducer now to handle this:

``` javascript
export const employeeReducer = (state = initialState, action) => {
    switch (action.type) {
        case REQUEST_EMPLOYEES_SUCCESS: {
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

For the moment I am not handling the case for `REQUEST_EMPLOYEES` because there isn't much to do there.
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
be `true`. So in my reducer in the case `REQUEST_EMPLOYEES` I will set `hasLoaded` to false,
and in the `REQUEST_EMPLOYEES_SUCCESS` i set it to true.

``` javascript
export const employeeReducer = (state = initialState, action) => {
    switch (action.type) {
        case REQUEST_EMPLOYEES: {
            return Object.assign({}, state, {
                hasLoaded: false
            })
        }
        case REQUEST_EMPLOYEES_SUCCESS: {
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
won't work any more now that we have turned `getEmployees` into an asyc call. We could do another asyncronousy
call to get the single employee but since the data is the same we might as well just share state and 
filter the one we want.

So I have added Redux `connect()` to `EmployeeProfile`, mapped employees to the props and filtering
the results as I was before:

``` javascript

```