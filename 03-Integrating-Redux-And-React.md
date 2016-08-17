# Integrating Redux and React

In the past few sections we have been working with Redux purely as a library. Now we are going
to get back to the Employee C.V Manager app that I started in the [React Introduction tutorial](https://github.com/justsayno/react-introduction-tutorial).

I have created a seperate repository for this tutorial and it can be found [here](https://github.com/justsayno/redux-react-introduction)
Once again you can clone the repository and follow the commands in each step to see the code
changes as you follow along. The branches correspond to the step in this
tutorial.

```
git checkout step-0
git clone https://github.com/justsayno/react-introduction-tutorial
```

Although Redux was born of React you can and I have used it with other frameworks. There is
a great library though called `react-redux` that you can use to wire up your Redux store
into the application seamlessly.

## Creating the store.

```
git checkout step-3
```

I have created a file `src/store.js` where I am going to keep all my Redux logic. In a larger
app I would break this up but I am trying to keep the example simple and understandable.

``` javascript
// store.js
import { createStore } from 'redux'
import { getEmployees } from './api/employees'

const initialState = {
    employee: getEmployees()
}

const employeeReducer = (state = initialState, action) => {
    return state
}

export const Store = createStore(employeeReducer)
```

This is very similar to what was in the last step except just for demonstration
I am passing the result of getEmployees() in on the initial state for now.

## Provider

```
git checkout step-3-1
```

If you look in `src/index.js` you will see I have used another import from Redux
and that is the `<Provider>` component.

While you could pass have references to your store all throughout your app and call `getState()` and `dispatch()` where
ever you want this is not a very scalable approach. It makes your components
tightly coupled to Redux and it makes it harder to test and reason about.

Redux's Provider is a [Higher Order Component](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#.cm8cecxk0)
 that helps with the task of mapping the state of your app to the props of your container components. 
 Don't worry if you don't understand that just now all you need to know is that the following code wraps 
 our app in a Redux provider make the store available to all container components.

``` javascript
ReactDOM.render(
  <Provider store={Store}>
    <Routes />
  </Provider>,
  document.getElementById('root')
)
```

## Map State to props

So far the app hasn't actually changed. I am still using getting my data by directly calling `getEmployees` in my 
`EmployeeDashboard`. I am going to use the other great feature of `react-redux` which is the `connect` function.

```
git checkout step-3-2
```

You *could* manually map all your state into the local state of your app but this has both performance and
testing downsides. You don't really want the component to be tightly coupled to the shape of the state
or Redux itself. I have changes `src/Containers/EmployeeDashboard.js` to the following:

``` javascript
import React, { Component } from 'react'
import { connect } from 'react-redux'

// Components
import EmployeeList from '../components/EmployeeList'
import EmployeeListItem from '../components/EmployeeListItem'

class EmployeeDashboard extends Component {
  render() {
    let { employees } = this.props
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

const mapStateToProps = (state) => ({
  employees: state.employees
})

export default connect(mapStateToProps)(EmployeeDashboard)
```

How connect works is that it takes two arguments (we are only using one at the moment) and returns a function
that you can pass your component to and 'connect' it to your Redux store.

**mapStateToProps**

This is a function that receives the current state and returns and object that will then be merged
with the other props the container component receives. This is how you manage the state in your app, when the Redux store changes it re-runs your mapStateToProps and passes
the changes through to the component via props.

**mapDispatchToProps**

This is a function that recieves the store's `dispatch` function and returns an object that will be merged with
the components props. I will cover this more later but it allows further abstraction of the inner workings 
of Redux away from your components.

## Consolidating our propTypes

```
git checkout step-3-3
```

I was going to add prop types to indicate what the EmployeeDashboard expects as 
props from the store (the employees array) but I didn't want to keep repeating the
PropType definition for an employee everywhere so I created a file `src/constatnts/PropTypes`
where I put my employee definition:

``` javascript
import { PropTypes } from 'react'
export const Employee = {
    id: PropTypes.string.isRequired,
    firstName: PropTypes.string.isRequired,
    lastName: PropTypes.string.isRequired,
    avatar: PropTypes.string.isRequired,
    role: PropTypes.string.isRequired,
    team: PropTypes.string.isRequired,
    biography: PropTypes.string.isRequired,
    keySkills: PropTypes.arrayOf(PropTypes.shape({
        name: PropTypes.string.isRequired
    })),
    recentProjects: PropTypes.arrayOf(PropTypes.shape({
        name: PropTypes.string.isRequired
    }))
}
```

Imported and used it like this:

```
//EmployeeDashboard
EmployeeDashboard.propTypes = {
    employees: PropTypes.arrayOf(PropTypes.shape(Employee)).isRequired
}
```

Now what about using an action to get the employee data?

[Next Step](4.Actions-And-Action-Creators)




