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

``` 
git checkout step-6-3
```

One thing we have glossed over is if the promise rejects... We can easily add another flag `hasError` to
our state:

``` javascript
const initialState = {
    employees: [],
    hasLoaded: false,
    isFetching: false,
    hasError: false
}
```

