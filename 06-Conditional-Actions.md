# Conditional Actions

Sometimes you want to put a little bit of logic into your action crators and potentially not call any action under
certain conditions. A good example of this is if there is more than one component that relies on the same
data from an API. 

A common approach is to extend the state to include details about the fetching of the data
as well. This means we update our initial state to look like this:

``` javascript
const initialState = {
    employees: [],
    hasLoaded: false,
    isFetching: false
}
```

Now I can use the `isFetching` property to store if the employee data is currently being fetched from an API (or include
this case my fake API).

```
git checkout step-6
```

If you look in our `src/store.js` file then you will see I've updated the initial state as well as the reducer:

``` javascript
export const employeeReducer = (state = initialState, action) => {
    switch (action.type) {
        case EMPLOYEES_REQUESTED: {
            return Object.assign({}, state, {
                hasLoaded: false,
                isFetching: true
            })
        }
        case EMPLOYEES_RECEIVED: {
            return Object.assign({}, state, {
                employees: action.employees,
                hasLoaded: true,
                isFetching: false
            })
        }
        default: {
            return state
        }
    }
}
```

I am setting `isFetching` to true when the action type is `EMPLOYEES_REQUESTED` and setting it back to false
when it is `EMPLOYEES_RECEIVED`. Now I can use this value in my action creator for `requestEmployees`.

```
git checkout step-6-1
```

So far we haven't used the second argument of the Redux Thunk function `getState`. If you open up `src/store.js`
now you will see the following:

``` javascript
export const requestEmployees = () => {
    return (dispatch, getState) => {
        const { hasLoaded, isFetching } = getState()
        if( hasLoaded || isFetching ) return
        
        dispatch(employeesRequested())
        return getEmployees().then(
            (employees) => dispatch(employeesReceived(employees))
        )
    };
}
```

We have pulled out the `hasLoaded` and `isFetching` from the current state and if either are true then we `bail out`
by returning. This will stop the dispatching of any actions and effectively do nothing.

We can now safely add this call to the `EmployeeProfile` components `componentWillMount` function and know that
if the data is already there it won't kick off another API call.

```
git checkouts step-6-2
```

I have basically added the same logic that is in the EmployeeDashbaord to conditionally display
the spinner based on the `isLoading` that is exposed through `mapStateToProps` as well
as calling the action creator `requestEmployees` in `willComponentMount`.

``` javasscript
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'

// PropTypes
import { Employee } from '../constants/PropTypes'

// Components
import Spinner from '../components/Spinner'

// Actions
import { requestEmployees } from '../store'

class EmployeeProfile extends Component {
    _getSelectedEmployee(props){
        // get employee and employee id from props
        const { employees, params: { employeeId } } = props
        
        // filter employees for the one that is selected
        return employees.filter((value) => {
            return value && (value.id === employeeId)
        })[0]
    }
    componentWillMount () {
        const { requestEmployees } = this.props
        requestEmployees()
    }
    render(){
        const { hasLoaded } = this.props
        if(!hasLoaded){
            return <Spinner />
        }
        // deconstruct the employee object for easier rendering
        const { firstName, lastName, role, team, biography, avatar, keySkills, recentProjects } = this._getSelectedEmployee(this.props)
        return (
            <div>
                //exluded for brevity
                {...}
            </div>
        )
    }
}


EmployeeProfile.propTypes = {
    employees: PropTypes.arrayOf(PropTypes.shape(Employee)).isRequired
}

const mapStateToProps = (state) => ({ 
    employees: state.employees,
    hasLoaded: state.hasLoaded
})

const mapDispatchToProps = (dispatch) => ({
  requestEmployees: () => dispatch(requestEmployees())
})

export default connect(mapStateToProps, mapDispatchToProps)(EmployeeProfile)

```

## Handling errors

One thing we have glossed over is if the promise is rejected. I'll fix that now. 

``` 
git checkout step-6-3
```

I added a new actions:

```
const EMPLOYEES_ERROR_RECEIVED = "EMPLOYEES_ERROR_RECEIVED"
```

Added another flag `hasError` and a key `error` to store any errors in our state:

``` javascript
const initialState = {
    employees: [],
    hasLoaded: false,
    isFetching: false,
    hasError: false,
    error: null
}
```

Created a handler for it in my reducer where the state indicated it has finisehd loading but there 
is an error. I've also updated the other handlers to reflect this change:

``` javascript
export const employeeReducer = (state = initialState, action) => {
    switch (action.type) {
        case EMPLOYEES_REQUESTED: {
            return Object.assign({}, state, {
                employees: [],
                hasLoaded: false,
                isFetching: true,
                hasError: false,
                error: null
            })
        }
        case EMPLOYEES_RECEIVED: {
            return Object.assign({}, state, {
                employees: action.employees,
                hasLoaded: true,
                isFetching: false,
                hasError: false,
                error: null
            })
        }
        case EMPLOYEES_ERROR_RECEIVED: {
            return Object.assign({}, state, {
                employees: [],
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
```

This will handle the action but we have not actually triggered it anywhere.

``` 
git checkout step-6-4
```

I have updated the `requestEmployees` function to inlude a callback being passed to my `then` as an
error handler. This simply passes the error along to an action creator that creates my action
with the type of `EMPLOYEES_ERROR_RECEIVED` and a property `error`.

``` javascript
export const employeesErrorReceived = (error) => ({
    type: EMPLOYEES_ERROR_RECEIVED,
    error: error
})

export const requestEmployees = () => {
    return (dispatch, getState) => {
        const { hasLoaded, isFetching } = getState()
        if( hasLoaded || isFetching ) return
        
        dispatch(employeesRequested())
        return getEmployees().then(
            (employees) => dispatch(employeesReceived(employees)),
            (error) => dispatch(employeesErrorReceived(error))
        )
    };
}
```

## Displaying the error

```
git checkout step-6-5
```

I've created a simple error component:

``` javascript
import React from 'react'

const Error = ({error}) => (
    <div>
        <p>Sorry something went wrong...</p>
        <p>{error}</p>
    </div>
)

export default Error
```

I have also updated both `EmployeeDashboard` and `EmployeeProfile` to include the error props and PropTypes
types as well as render the error component if the `hasError` flag is true. For example the `EmployeeProfile` component:

``` javascript
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'

// PropTypes
import { Employee } from '../constants/PropTypes'

// Components
import Spinner from '../components/Spinner'
import Error from '../components/Error'

// Actions
import { requestEmployees } from '../store'

class EmployeeProfile extends Component {
    _getSelectedEmployee(props){
        // get employee and employee id from props
        const { employees, params: { employeeId } } = props
        
        // filter employees for the one that is selected
        return employees.filter((value) => {
            return value && (value.id === employeeId)
        })[0]
    }
    componentWillMount () {
        const { requestEmployees } = this.props
        requestEmployees()
    }
    render(){
        const { hasLoaded } = this.props
        if(!hasLoaded){
            return <Spinner />
        }

        const { hasError, error } = this.props
        if(hasError){
            return <Error error={error} />
        }

        // deconstruct the employee object for easier rendering
        const { firstName, lastName, role, team, biography, avatar, keySkills, recentProjects } = this._getSelectedEmployee(this.props)
        return (
            <div>
               {...}
            </div>
        )
    }
}


EmployeeProfile.propTypes = {
    requestEmployees: PropTypes.func.isRequired,
    employees: PropTypes.arrayOf(PropTypes.shape(Employee)).isRequired,
    hasLoaded: PropTypes.bool.isRequired,
    hasError: PropTypes.bool.isRequired,
    error: PropTypes.string
}

const mapStateToProps = (state) => ({ 
    employees: state.employees,
    hasLoaded: state.hasLoaded,
    hasError: state.hasError,
    error: state.error
})

const mapDispatchToProps = (dispatch) => ({
  requestEmployees: () => dispatch(requestEmployees())
})

export default connect(mapStateToProps, mapDispatchToProps)(EmployeeProfile)

```

But we don't have a real error to show so I'm going to fake it by rejecting in my fake API service

```
git checkout step-6-6
```

Now if you reload the page you will get the error message coming up saing 'Could not load employees'.
