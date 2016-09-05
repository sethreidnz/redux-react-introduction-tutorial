# Combine Reducders

```
git checkout step-7
```

In the current implementation of our application the data used for the single view and the list view of the employees is the same. I've changed this now
so that the list doesn't have as much detail so we will have to get the employee details when the user navigates there...

What I have done in this stage is three things:

1. Change the mock API so that there are two different calls
2. Added a new PropType called EmployeeSimplified

```
export const EmployeeSimlified = {
    id: PropTypes.string.isRequired,
    firstName: PropTypes.string.isRequired,
    lastName: PropTypes.string.isRequired,
    avatar: PropTypes.string.isRequired,
    role: PropTypes.string.isRequired,
    team: PropTypes.string.isRequired
}
```

3. Updated each of my 'list' view components to use the new `EmployeeSimlified` PropType to avoid any errors.

But now my `EmployeeProfile` view is broken since the data doesn't have all the fields it needs! Lets fix this by adding
our new redux actions and action handler.

## Multiple Reducers

Normally at this stage I would seperate things into different files based on the [Fractal Project Structure](https://github.com/davezuko/react-redux-starter-kit/wiki/Fractal-Project-Structure)
but I am leaving that for a later tutorial. For now I am going to leave everything in one file for clarity.

But I am going to add a new branch on our state tree. So far all the data has been at the root level like this:

``` javascript
const initialState = {
    employees: [],
    hasLoaded: false,
    isFetching: false,
    hasError: false,
    error: null
}
```

But now we want to store seperate data about each individual employee. You could find the employee you have viewed and add the extra data into the list, but this gets hard to manage.
Instread Redux gives you a method called `combineReducers` which means we can make multiple reducers and pass them to our `createStore` function as what appears to be one reducer.

Each reducer manages a seperate part of the state. For example:

``` javascript
const employeesInitialState = {
    employees: [],
    hasLoaded: false,
    isFetching: false,
    hasError: false,
    error: null
}ror: null
}

const selectedEmployeeInitialState = {
    employee: null,
    hasLoaded: false,
    isFetching: false,
    hasError: false,
    error: null
}
```

```
git checkout step-7-1
```

In this step I have added all the required code to create the actions, action creators, the seperate reducers and using combine reducers to bring it all together.
It might seem like a lot of code but in my more advanced tutorial that will follow this one I will show you how you can generalise a lot of this code to reduce the boilerplate.

``` javascript
import { createStore, applyMiddleware, compose, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import { getEmployees, getEmployee } from './api/employees'

// Actions
const EMPLOYEES_REQUESTED = 'EMPLOYEES_REQUESTED'
const EMPLOYEES_RECEIVED = "EMPLOYEES_RECEIVED"
const EMPLOYEES_ERROR_RECEIVED = "EMPLOYEES_ERROR_RECEIVED"

const EMPLOYEE_SELECTED = 'EMPLOYEE_SELECTED'
const EMPLOYEE_RECEIVED = "EMPLOYEE_RECEIVED"
const EMPLOYEE_ERROR_RECEIVED = "EMPLOYEE_ERROR_RECEIVED"

// employees action creators
export const employeesRequested = () => ({
    type: EMPLOYEES_REQUESTED
})

export const employeesReceived = (employees) => ({
    type: EMPLOYEES_RECEIVED,
    employees: employees
})

export const employeesErrorReceived = (error) => ({
    type: EMPLOYEES_ERROR_RECEIVED,
    error: error
})

export const requestEmployees = () => {
    return (dispatch, getState) => {
        const state = getState()
        const { hasLoaded, isFetching } = state.employees
        if( hasLoaded || isFetching ) return
        
        dispatch(employeesRequested())
        return getEmployees().then(
            (employees) => dispatch(employeesReceived(employees)),
            (error) => dispatch(employeesErrorReceived(error))
        )
    }
}

// selected employee action creators
export const employeeSelected = (employeeId) => ({
    type: EMPLOYEE_SELECTED,
    employeeId: employeeId
})

export const employeeReceived = (employee) => ({
    type: EMPLOYEE_RECEIVED,
    employee: employee
})

export const employeeErrorReceived = (error) => ({
    type: EMPLOYEE_ERROR_RECEIVED,
    error: error
})

export const selectEmployee = (employeeId) => {
    return (dispatch, getState) => {
        debugger
        const state = getState()
        const { hasLoaded, isFetching } = state.selectedEmployee
        if( hasLoaded || isFetching ) return
        
        dispatch(employeeSelected(employeeId))
        return getEmployee(employeeId).then(
            (employee) => dispatch(employeeReceived(employee)),
            (error) => dispatch(employeeErrorReceived(error))
        )
    }
}

// initial state of the employee reducer
const employeesInitialState = {
    items: [],
    hasLoaded: false,
    isFetching: false,
    hasError: false,
    error: null
}

// employees reducer
export const employeeReducer = (state = employeesInitialState, action) => {
    switch (action.type) {
        case EMPLOYEES_REQUESTED: {
            return Object.assign({}, state, {
                items: [],
                hasLoaded: false,
                isFetching: true,
                hasError: false,
                error: null
            })
        }
        case EMPLOYEES_RECEIVED: {
            return Object.assign({}, state, {
                items: action.employees,
                hasLoaded: true,
                isFetching: false,
                hasError: false,
                error: null
            })
        }
        case EMPLOYEES_ERROR_RECEIVED: {
            return Object.assign({}, state, {
                items: [],
                hasLoaded: true,
                isFetching: false,
                hasError: true,
                error: action.error
            })
        }
        default: {
            return state
        }
    }
}

// initial state of the selected employee reducer
const selectedEmployeeInitialState = {
    item: null,
    hasLoaded: false,
    isFetching: false,
    hasError: false,
    error: null
}

// selected employee reducer
export const selectedEmployeeReducer = (state = selectedEmployeeInitialState, action) => {
    switch (action.type) {
        case EMPLOYEE_SELECTED: {
            return Object.assign({}, state, {
                item: null,
                hasLoaded: false,
                isFetching: true,
                hasError: false,
                error: null
            })
        }
        case EMPLOYEE_RECEIVED: {
            return Object.assign({}, state, {
                item: action.employee,
                hasLoaded: true,
                isFetching: false,
                hasError: false,
                error: null
            })
        }
        case EMPLOYEE_ERROR_RECEIVED: {
            return Object.assign({}, state, {
                item: null,
                hasLoaded: true,
                isFetching: false,
                hasError: true,
                error: action.error
            })
        }
        default: {
            return state
        }
    }
}

const rootReducer = combineReducers({
    employees: employeeReducer,
    selectedEmployee: selectedEmployeeReducer
})

const initialState = {
    employees: employeesInitialState,
    selectedEmployee: selectedEmployeeInitialState
}

export const store = createStore(
  rootReducer,
  initialState,
  compose(
    applyMiddleware(thunk),
    window.devToolsExtension ? window.devToolsExtension() : f => f
    )
)
```

Lets call this new `selectEmployee` action from our `EmployeeProfile` component to get it working again.

```
git checkout step-7-2
```

I have now changed my EmployeeProfile to remove the logic for getting the selected employee from the state to instaed call `selectEmployee` action creator:

``` javascript
componentWillMount () {
    const { selectEmployee, params: { employeeId }  } = this.props
    selectEmployee(employeeId)
}
```

Otherwise this is very similar.

## Dealing with stale state.

The only problem now is that once an employee is selected it doesn't stays selected. We neeed to invalidated the selected employee when the `employeeId` param 
changes. We can easily do this using router action provided by `redux-react-router` as well as our current implementation. But so far we haven't integrated our router into
our state...

> NOTE: TO BE COMPLETED

