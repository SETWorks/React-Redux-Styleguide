# React/Redux Style Guide

Inspired from [Dave Atchley - Gist](https://gist.github.com/datchley/4e0d05c526d532d1b05bf9b48b174faf)

## Table of Contents:
 
* [React/Redux Style Guide](#reactredux-style-guide)
  * [React Guidelines](#react-guidelines)
      * [Organization](#organization)
      * [Components](#components)
      * [JSX](#jsx)
      * [Component Styles](#component-styles)
      * [Naming Things](#naming-things)
  * [Redux](#redux)
      * [Organization](#organization-1)
      * [Action Creators](#action-creators)
      * [Reducers](#reducers)
      * [Connecting Components](#connecting-components)
      * [Organizing Redux State](#organizing-redux-state)
      * [Naming Things](#naming-things-1)
  * [Utility/Helper Functions](#utilityhelper-functions)



This is a working set of guidelines for developing React applications.

# What is this for?

- SETWorks Devs

The guide is meant to provide direction on building quality code across the UI and the development team.  


## React Guidelines
React makes it easy to build modular, declarative, components. 

#### Useful Resources:
- [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) - *Facebook React Docs*
- [You're missing the point of React](https://medium.com/@dan_abramov/youre-missing-the-point-of-react-a20e34a51e1a) - by *Dan Abramov*
- [Best Practices for building large React apps](https://engineering.siftscience.com/best-practices-for-building-large-react-applications/) - by *Alex Lopatin*
- [Atomic Design](http://bradfrost.com/blog/post/atomic-web-design/) - by *Brad Frost* (*not React specific, but applicable*)

 
###  Organization

#### Useful Resources:
- [How to Scale React Applications](https://www.smashingmagazine.com/2016/09/how-to-scale-react-applications/) - by *Max Stoiber*
- [Smart &amp; Dumb Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.74ous68x2) -  by *Dan Abramov*
- [Organizing Large React Applications](http://engineering.kapost.com/2016/01/organizing-large-react-applications/) - by *Nathanael Beisiegel*
- [Feature First Organization](https://medium.com/front-end-hacking/the-secret-to-organization-in-functional-programming-913484e85fc9#.hntdncupt) - by *Ryan C. Collins*



<a name="react-organization--component_folders"></a><a name="1.3"></a> 
§ [1.3](#react-organization--component_folders) **Component Folder Organization**: We a


**[⬆ back to top](#table-of-contents)**


### Components
Useful Resources:
- [React Patterns](http://reactpatterns.com/) - by *Michael Chan*
- [A Conceptual Intro to React Components](https://ifelse.io/2016/10/20/introducing-react-components/) - by *Mark Thomas*
- [Higher Order Components: React Pattern](https://www.sitepoint.com/react-higher-order-components/) - by *Jack Franklin*


<a name="react-components--keep_small"></a><a name="2.1"></a> 
§ [2.1](#react-components--keep_small) **Keep Components Small**: Keeping components small maximizes their potential for reuse, reduces the chance for bugs, allows them to focus on a single reponsibility and improves readability and testing.

> Why? Rule of thumb is that if your render method has more than 10-20 lines it's probably way too big. The whole idea behind React is code re-usability (*separation of concerns, single responsibility principle, declarative*) so if you just throw everything in one file you are losing clarity and simplicity.

<a name="react-components--one_per_file"></a><a name="2.2"></a> 
§ [2.2](#react-components--one_per_file) **One Component per file**: You should try to put each component in its own file to ensure they are easier to read, maintain and test.

> Why? This follows from [2.1](#2.1) to help in keeping components focused while being easier to read and test. If you're unsure where a larger component might be better broken out into smaller components, it's ok to break out smaller components together in the same file; you can come back and refactor as the requirements/design become more clear and pull them out into separate files then.

<a name="react-components--use_composition"></a><a name="2.3"></a> 
§ [2.3](#react-components--use_composition) **Use Composition to extend functionality**: Use composition and Higher Order Components (HoC's) to extend other components or add functionality.

- **Composition** of components is handled via `this.props.children`, allowing one component to render the output of one or more components by containing them as nested components.
```jsx
{/* Composition via children */}
<Modal show={true}>
  <ModalTitle>My Modal</ModalTitle>
  <ModalBody>
	  Welcome to composition!
  </ModalBody>
</Modal>
```
- A **higher-order component** (*HoC*) is just a function that takes an existing component and returns another component that wraps it. The HoC could do any of the following:
	- Do things before and/or after it calls the wrapped component
	- Avoid rendering the wrapped component if certain criteria is not met
	- Update the props passed to the wrapped component, or add new props
	- Transform the output of rendering the wrapped component (*e.g. wrap with extra DOM elements, etc.*)

- Example - React Timer using HoC ([live demo](http://codepen.io/datchley/pen/xRgdKL?editors=0010))

```jsx
// on-interval.jsx
// HoC to allow us to refresh a component 
// based on a timing interval 
const onInterval = (refresh) => (WrappedComponent) => {
  return class WithInterval extends Component {
	constructor(props) {
      super(props);
	  this.state = { ticks: 0 };
	  this.interval = setInterval(this.tick.bind(this), refresh);
	}
    
    tick() {
      this.setState({ ticks: this.state.ticks + 1 })
    }
    
    componentWillUnmount() {
	  clearInterval(this.interval);
	}
    
    render() {
      return <WrappedComponent {...this.props} />;
	}
  };
};
export default onInterval;
```
```jsx
// timer.js
// Component to display current time HH:MM:SS A/PM
import onInterval from './on-interval';

const Timer = ({ label }) => {
  const now = new Date();
  const hours = (now.getHours() % 12) || 12;
  const mins = now.getMinutes();
  const secs = now.getSeconds();
  return (<p>
    <b>{label && `${label}: `}</b> 
    {(hours < 10 ? '0' : '') + hours}:
    {(mins < 10 ? '0' : '') + mins}:
    {(secs < 10 ? '0' : '') + secs}
    {hours < 12 ? ' PM' : ' AM'}
  </p>);
};

export default onInterval(1000)(Timer);
```

```jsx
// app.js
import Timer from './timer';
import { render } from 'react-dom';

render(<Timer label="Current Time"/>, document.querySelector('#app'));

// Output, updating every second
// => Current Time: 12:35:28 AM
// => Current Time: 12:35:29 AM
// => ...
```

<a name="react-components--declare_statics"></a><a name="2.4"></a> 
§ [2.4](#react-components--declare_statics) **Always declare `propTypes`, `defaultProps` and `displayName`**: Always declare prop types and display name for all components (*container or presentational*); and declare default props for any non-required props. 

> Why? propTypes are a form of documentation, and providing defaultProps means the reader of your code doesn’t have to assume as much. In addition, it can mean that your code can omit certain type checks.

```jsx
// bad
function Bookend({ left, right, children }) {
  return <div>{foo}{children}{bar}</div>;
}
Bookend.propTypes = {
  left: PropTypes.number.isRequired,
  right: PropTypes.string
};

// good
function Bookend({ left, right, children }) {
  return <div>{left}{children}{right}</div>;
}
Bookend.displayName = 'Bookend';
Bookend.propTypes = {
  left: PropTypes.string.isRequired,
  right: PropTypes.string,
  children: PropTypes.node
};
Bookend.defaultProps = {
  right: '',
  children: null,
}; 
```
- default props on functional components can also be handled using ES2015 default argument values.

```es6
function Bookend({ left, right = '', children = null }) { ... }
```

**[⬆ back to top](#table-of-contents)**

### JSX

<a name="react-jsx--keep_simple"></a><a name="5.1"></a> 
§ [5.1](#react-jsx--keep_simple) **Keep conditional rendering above the return statement**: Handle conditional rendering in your component by using one of the following techniques to help keep code clean, readable and easy to extend:

- Using `if..else` outside of JSX, or a ternary `?:` inline w/ the JSX

```jsx
// good
render() {
  if (this.props.username) {
    return <UserGreeting user={this.props.username}/>;
  }
  else {
    return <GuestGreeting />;
  }
}
```

```jsx
// good 
render() {
  return this.props.username ?
    <UserGreeting user={this.props.username} /> :
    <GuestGreeting>;
}
```

- if the condition is complex or it clutters the `render()` logic, pull it out into a separate method.

```jsx
class Header extends Component {
  isValidUser({ user } = this.props) {
    return user.isLoggedIn && user.hasEntitlement('ADMIN');
  }
  render() {
    const { user } = this.props;
    return (
      <div>
        <h1>{title}</h1>
        <div className="pull-right">
          { this.isValidUser(user) ?
              <UserNavProfile user={user}/> :
              <GuestNavProfile />
           }
        </div>
      </div>
    );
  }
}  
```

- Simple conditional tests using `&&`

```jsx
// good
render() {
  const { error } = this.props;
  return (
    <p>
      { error && <Notification message={error}/> }
    </p>
  );
}
```
> **Tip:** When using conditional expressions in JSX, be sure your conditional expression evaluates to a one of: `false`, `null`, `undefined`, and `true` which are values ignored by React for rendering. Some falsy values, like the number `0` (*zero*), are still rendered by React, so the following would actually output the string `0` for the conditional expression if the `records` array had zero length:

```javascript
// bad
const Rows = ({ records }) => ( 
  <div>
    { records.length && records.map(/*...*/) }
  </div>
);
```

> **Tip:** If a condition becomes too complex or cumbersome, it might be time to pull that portion out into a new, smaller component to encapsulate that logic. 

<a name="react-jsx--use_map_for_lists"></a><a name="5.2"></a> 
§ [5.2](#react-jsx--use_map_for_lists) **Use `map` to output/render a list of items**: JSX allows [any valid javascript expression](https://facebook.github.io/react/docs/jsx-in-depth.html#javascript-expressions) within `{}` curly braces, which allows you to use functions like `Array#map` to loop over lists and return an array of valid JSX elements.


> Why? Keep in mind that React/JSX will attempt to render the result of any valid javascript expression that doesn't result in one of these values:
> 
- `false`
- `null`
- `undefined`
- `true`
> 
> So, returning any other value, including JSX, or an array of any of the above, will get rendered. Since `map()`, and even related methods like `filter()` and `reduce()` return arrays, they can all be used to iterate over lists inline with JSX.


**[⬆ back to top](#table-of-contents)**

### Component Styles
- TODO

**[⬆ back to top](#table-of-contents)**

### Naming Things

<a name="react-naming--consistent_naming"></a><a name="7.1"></a> 
§ [7.1](#react-naming--consistent_naming) **Use consistent naming for events and event handlers**: Name the handler methods after their triggering event.

Event handlers should:

- begin with `handle` 
- end with the name of the event they handle (eg, `Click`, `Change`) 
- be present-tense
```javascript
// bad
punchABadger () { /*...*/ },

render () {
  return <div onClick={this.punchABadger} />;
}
```

```javascript
// good
handleClick () { /*...*/ },

render () {
  return <div onClick={this.handleClick} />;
}
```

Event names:

- in props should start with `on`
- should not clash with builtin/native event names, eg. use `onSelect` instead of `onFocus` or `onClick`

```javascript
handleClick() { 
  const { onFlip } = this.props;
  onFlip(this.state.flipped);
}

render() {
  return (
    <div onClick={this.handleClick}>
      { this.props.children }
    </div>
  );
}
```

> **Tip:** If you need to disambiguate handlers or event names, add additional information between `handle` or `on` and the event name. For example, you can distinguish between `onChange` handlers: `handleNameChange` and `handleAgeChange` and use props `onNameChange` and `onAgeChange`, respectively.
>
> When you do this, ask yourself if you should be creating a new component.

**[⬆ back to top](#table-of-contents)**


## Redux

Useful Resources:
- [Redux Docs](http://redux.js.org/index.html)
- [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) - (*video series*) by *Dan Abramov*
- [Idiomatic Redux](https://egghead.io/series/building-react-applications-with-idiomatic-redux) - (*video series*) by *Dan Abramov*
- [A maintainable project struture for React/Redux](https://hackernoon.com/my-journey-toward-a-maintainable-project-structure-for-react-redux-b05dfd999b5#.113ysmtbt) - by *Matteo Mazzarolo*

### Organization

<a name="redux-organization--feature_first_ducks"></a><a name="8.1"></a> 
§ [8.1](#redux-organization--feature_first_ducks) **Co-locate reducers, actions, action-types and selectors (*similar to [ducks](https://github.com/erikras/ducks-modular-redux) standard*)**: Organizing your redux related code around the reducer (*the slice of store state it manages*) by bundling your `actions`, `action types` w/ the `reducer` helps organize your redux code into reusable modules, is clean, reduces unnecessary boilerplate and eases developer effort by co-locating.

> **Tip:** Our suggested format is a slight modification of the [DUCKS](https://github.com/erikras/ducks-modular-redux) standard which is widely adopted. Instead of including the reducer, actions, types in a single file, we separate them to ease usage/importing and keep them in a shared folder for the module.

A redux module (*bundled folder*) can be organized as follows:

```
src/redux-modules/
  authorization/
    index.js     # default exports reducer
    actions.js   # named exports of action creators
    types.js     # exports action type constants
    selectors.js # exports selectors
```
- Example:
```javascript
// src/redux-modules/auth/index.js
import * as types from './types';

// initial state for reducer
const initialState = { /* ... */ };

// reducer
export default (state = initialState, action) => {
  switch (action.type) {
    case types.ACTION_TYPE: return {
        ...state,
        /* modify state in immutable fashion */
      };
    /* other ACTION_TYPEs... */
    default: 
      return state;
  }
};
```
```javascript
// src/redux-modules/auth/types.js

// named action type constants
export const ACTION_TYPE_ONE = '@@auth/action-type-one';
export const ACTION_TYPE_TWO = '@@auth/action-type-two';
/* ... */
```

```javascript
// src/redux-modules/auth/actions.js
import * as types from './types';

export const actionCreator = (/* params... */) => ({
  type: types.ACTION_TYPE_ONE,
  /* params... */
});

/* other action creators... */
```

> Why? This helps us build modules that:
> 
> - focused on the managing a slice of application state, making for better reuse across pages/components
> - keeps us from coupling redux state w/ container components
> - gives us an easy way to import reducers, as well as action creators across components, and action types across other redux modules if needed.
> 
> Grouping by file type, ie. `src/actions`, `src/reducers`, `src/action-types`, `src/selectors`, etc. doesn't scale well for large applications, is less reusable because files related to a slice of state are spread out over the filesystem; and making changes means developers end up having to edit multiple files all over the filesystem as well.

For example...
Importing reducer(s) is easy:
```javascript
import authReducer from 'redux-modules/authorization';
import otherReducer from 'redux-modules/other';

const reducer = combineReducers({
  authReducer,
  otherReducer
});
```
Reusing actions across components:
```javascript
// some-component.js
import * as authActions from 'redux-modules/authorization/actions';
import * as otherActions from 'redux-modules/other/actions';

// ...
const mapDispatchToProps = (dispatch) => ({
  ...bindActionCreators({
    ...authActions,
    ...otherActions
  }, dispatch)
});
// ...
```
Listening to actions across reducers:
```javascript
// redux-modules/some-other/index.js
import * as authTypes from 'redux-modules/auth/types';
import * as types from './types';

// reducer...
export default (state = {}, action) => {
  case authTypes.OUTSIDE_ACTION:
    /* respond to OUTSIDE_ACTION somehow ... */
  case types.OWN_ACTION:
    /* handle our own actions ... */
  // ...
};
```

<a name="redux-organization--colocate_unit_tests"></a><a name="8.2"></a> 
§ [8.2](#redux-organization--colocate_unit_tests) **Related to [8.1](#8.1) colocate your unit tests for your reducer and actions with the redux-module**: Since we're bundling our redux code to be reusable as modules, keeping the unit tests for each module colocated with the bundle makes sense. 

> Why? Similar to the feature-first organization for components and the reasons for our previous rule, with the unit tests bundled, this redux-module folder could easily be dropped into another project "as is" or easily pulled out into an `npm` module for wider reuse.
>
> For clear ideas on testing redux reducers and actions, see [Writing Tests](http://redux.js.org/docs/recipes/WritingTests.html) in the Redux docs and [Testing Redux Applications](http://randycoulman.com/blog/2016/03/15/testing-redux-applications/) by *Randy Coulman*

**[⬆ back to top](#table-of-contents)**


### Action Creators

<a name="redux-action-creators--async_actions"></a><a name="9.1"></a> 
§ [9.1](#redux-action-creators--async_actions) **Use `redux-thunk` and `async`/`await` for asynchronous action creators** As a first resort, using the `redux-thunk` middleware, along with taking advantage of `async`/`await` is a clean approach to organizing asynchronous action creators.

> Why? Using the redux-thunk middleware allows you to create asynchronous actions in a manageable way. You can use other potential redux middleware for this as well, like `redux-promise` or `redux-sagas`; but using thunks will likely handle most of your needs.
>
> Taking advantage of ES2016's `async`/`await` syntax when doing async `fetch` request will also make your code cleaner and easier to follow vs using Promises and chained/nested `.then()` handlers.  The more complex the response to the various sequence of actions needs to be, the more cumbersome, nested and unreadable the Promise chain tends to be. Using `async` and `await` allows you to write asynchronous code the reads like it was synchronous.

```javascript
// bad
const complexAction = (/*...*/) => (dispatch) => {
  dispatch(doRequestAction(/*...*/));
  fetch(/*...*/)
  .then(handleResponse)
  .then(
    json => dispatch(doSuccessAction(json)),
    error => dispatch(doErrorAction(json))
  );
};

// good
const complextAction = (/*...*/) => async (dispatch) => {
  dispatch(doRequestAction(/*...*/);
  try {
    const json = await fetch(/*...*/);
    dispatch(doSuccessAction(/*...*/);
  }
  catch (err) {
    dispatch(doErrorAction(/*...*/);
  }
};
```

<a name="redux-action-creators--no_access_state_in_action"></a><a name="9.2"></a> 
§ [9.2](#redux-action-creators--no_access_state_in_action) **Don't attempt to access store state from async `redux-thunk` action creators** When using `redux-thunk`, the returned function is passed two parameters, 1) the `dispatch` method from the store, and 2) the `getState` method of the store. To keep action creators, and their redux modules decoupled from the application's store structure it's best to pass needed state as props to the action creator rather than use `getState()`.

It's tempting to move conditional logic into the action creators when using `redux-thunk` because of the availability of the store's `getState` method as the second param:

```javascript
// brittle
const loadTodo = (id) => (dispatch, getState) => {
  // only fetch the todo if it isn't already loaded
  if (!getState().todos.includes(id)) {
    const todo = await fetch(`/todos/${id}`);
    dispatch(addTodo(todo));
  }
}
```

> Why? This kind of state-dependent logic in the action creator is brittle because the action creator is now tied to the global state shape decided by the application. If the application decided to change the state shape (*ie., nested reducers in `combineReducers` or changed the state key the todos reducer is assigned to*) this action creator would break.  This problem is compounded if you have multiple action creators all relying on a specific state shape and key names.

A better approach is to do one of the following:

1. move the determination to call an action based on state to a container component, making the check in `componentDidMount()`, `componentWillReceiveProps()`, or appropriate event handler on container:
	```jsx
	// good
	// redux-module/actions
	const loadTodo = (id) => (dispatch) => {
	  const todo = await fetch(`/todos/${id}`);
	  dispatch(addTodo(todo));
	};
	
	// src/containers/Container
	@connect(
	  (state) => ({ todos: selectTodos(state) }),
	  (dispatch) => { loadTodo }
	)
	class Container extends Component {
		handleClick(id) {
		  const { todos, loadTodo } = this.props;
		  if (!todos.includes(id)) {
		    loadTodo(id);
		  }
		}
		//...
	}
	```
2. pass the required state or props needed for the conditional logic to the action creator as params:

	```javascript
	// good
	const loadTodo = (id, todos) => (dispatch) => {
	  // only fetch the todo if it isn't already loaded
	  if (!todos.includes(id)) {
	    const todo = await fetch(`/todos/${id}`);
	    dispatch(addTodo(todo));
	  }
	}
	```
3. or, if the logic in question is being used by multiple components, you can dynamically access the correct state slice by passing the state key to access rather than the actual state (*as in the last example*), allowing us to keep the logic in one place and not duplicate across components:
	
	```javascript
	// use lodash's `_.get` to create a generic selector that
	// can be used to access dynamic paths within a state object
	// https://lodash.com/docs/4.17.2#get
	const getStateIn = (state, path) => _.get(state, path);
	
	const loadTodo = (id, todosPath) => (dispatch, getState) => {
	  // only fetch the todo if it isn't already loaded
	  if (!getStateIn(getState(), todosPath).includes(id)) {
	    const todo = await fetch(`/todos/${id}`);
	    dispatch(addTodo(todo));
	  }
	}
	
	// given state shape of
	// { 
	//    collections: {
	//      todos: [],
	//      contacts: [],
	//    }
	// }
	loadTodos(4, 'collections.todos');
	```
	In the above implementation, the application can change the shape of the state and as long as it passes the correct key path, the action creators stay decoupled and don't have to be modified.
	
> **Tip:** Note that **#1** or **#3** above can run into problems when relying on `getState()` if another async action is dispatched prior to accessing the state.  `dispatch` calls are synchronous; but when dispatching another async action, the state might get modified before or after the use of `getState()`, which still makes state access brittle and prone to hard to debug errors.

**[⬆ back to top](#table-of-contents)**


### Reducers

<a name="redux-reducers_state_in_action"></a><a name="10.1"></a> 
§ [10.1](#redux-reducers--no_access_state_in_action) **Reducers can't be reused without implementing Higher Order Reducers** In order to reuse reducer logic in your store (*or have multiple instances of a connected component*) you'll need to use a higher order reducer to create instance specific reducers for each slice of state.

- it's tempting to think that just reusing a reducer is simple. Consider a simple set of counters:

	```javascript
	// reducer
	const counter = (state = 0, action) => {
	  switch(action.type) {
	    case INC: return state + 1;
	    case DEC: return state - 1;
	    default: return state;
	  }
	}

	const rootReducer = combineReducers({
		counterA: counter,
		counterB: counter,
		counterC: counter
	});
	```
	However, after dispatching an `INC` action, all three counters would be equal to `1`. This is because `combineReducers` calls each slice-reducer with the same action.

To get this to work correctly, you'll need to create a higher order reducer (*a reducer factory, essentially*) that takes a reducer as an argument and returns a new reducer that modifies the behavior of the passed reducer.  

We can do this in a generic fashion by creating a higher order reducer that takes a reducer and a predicate function for checking the action. If the predicate function returns `true`, it will pass the action on to the original reducer; otherwise, it just returns the state "as is."

```javascript
// higher-order reducer, to limit a reducer to work 
// on actions matching a predicate only
const limited = (reducer, predicate) => (state, action) => {
  if (predicate(action)) {
    return reducer(state, action);
  }
  return state;
};

const rootReducer = combineReducers({
	counterA: limited(counter, action => action.name == 'A'),
	counterB: limited(counter, action => action.name == 'B'),
	counterC: limited(counter, action => action.name == 'C')
});

// Now, just dispatch actions with an added 
// `name` property to identify the proper counter.
dispatch({ type: INC, name: 'A' });
dispatch({ type: DEC, name: 'C' });
// => new state
//  {
//    counterA: 1,
//    counterB: 0,
//    counterC: -1
//  }
```

<a name="redux-reducers--string_constant_action_types"></a><a name="10.2"></a> 
§ [10.2](#redux-reducers--string_constant_action_types) **Use string constants instead of inline strings for action types** There are a number of benefits to defining action types as string constants, rather than as inline strings in the reducers and action creators.


 - It helps keep the naming consistent, and allows you to gather all action types in one place. (*could be in same file as reducer, actions or in separate file*)
 - It's helpful to see all the existing actions before working on a new feature. Maybe someone added the action you need already. 
 - Easy to see changes and scope of features on pull requests by visually diffing new/removed/changed action types. 
 - You'll get `undefined` if you mistype a constant. Redux will immediately throw when dispatching such an action, and you'll find the mistake sooner.


```javascript
// bad
const reducer(state = {}, action) {
  switch(action.type) {
    case 'DO_STUFF': //...
    case 'GET_STUFF': //...
    //...
  }
}
const doStuff = () => ({ type: 'DO_STUFF', /*...*/ });
const getStuff = () => ({ type: 'GET_STUFF', /*...*/ });
```

```javascript
// good
const DO_STUFF = '@@stuff/do-it';
const GET_STUFF = '@@stuff/get-it';

const reducer(state = {}, action) {
  switch(action.type) {
    case DO_STUFF: //...
    case GET_STUFF: //...
    //...
  }
}
const doStuff = () => ({ type: DO_STUFF, /*...*/ });
const getStuff = () => ({ type: GET_STUFF, /*...*/ });
```

> **Tip:** Using a prefix for the action type based on the reducer/actions they work (*the `@@stuff/*` above*) with is a good way to namespace your action types and ensure you don't get any collisions across reducers when using simple names like `ADD_RECORD` or `GET_TODO` and end up with an unstable state.

<a name="redux-reducers--use_selectors_colocate"></a><a name="10.2"></a> 
§ [10.2](#redux-reducers--use_selectors_colocate) **Use selectors liberally, and co-locate selectors with their reducer** Reducers represent a slice of your data store, including it's "shape", and selectors are the query language for accessing that state at the component level.  Keep them together and use them to decouple your state model from your view.

If components only use selectors to access state, typically via a call to `mapStateToProps()` when using `react-redux`, you've effectively decoupled your view from your state model and logic.

```javascript
// bad
const initialState = { 
  step: 1,
  counter: 0 
};

const reducer(state = initialState, action) { /*...*/ }

// View Component
@connect((state) => ({ 
	count: state.counter
	nextCount: state.counter + state.step
})
//...
```

Here, if we change our reducer or state shape, any component accessing that state directly will break.  Using state directly, as above, tightly couples your components to your redux store.

```javascript
// good
const initialState = { 
  step: 1,
  counter: 0 
};

const reducer(state = initialState, action) { /*...*/ }

// simple selectors
const selectCount = (state) => state.counter;
const selectNextCount = (state) => state.counter + state.step;

// View Component
@connect((state) => ({ 
	count: selectCount(state),
	nextCount: selectNextCount(state)
})
//...
```
In this example, if we rename our state key from `counter` to `count`, any component using our selectors to access state still works without changes.  We are decoupled and can make changes to just the reducer/selectors.



<a name="redux-reducers--keep_reducers_pure"></a><a name="10.3"></a> 
§ [10.3](#redux-reducers--keep_reducers_pure) **Keep reducers pure, with no side-effects** Reducers should be pure functions, *ie.* deterministic. Redux works on the assumption that your state is immutable; and a reducer is intended to accept a `state`<sub>1</sub>, along with an `action` and return a completely new `state`<sub>2</sub> (*or the exact same state if nothing has changed*).


> <center>
>  **reducer** `=>` ( **state**<sub>cur</sub>,  **action** ) `=>` **state**<sub>next</sub>
> </center>
>
> A pure function is a function where the return value is only determined by its input values, without observable side effects.

Side effects like ajax calls, logging, dom updates, etc, should **__never__** be done within a reducer.

<a name="redux-reducers--keep_state_flat"></a><a name="10.4"></a> 
§ [10.4](#redux-reducers--keep_state_flat) **Try to keep your state shape flat; and normalize data where possible** [TODO]...


**[⬆ back to top](#table-of-contents)**


### Connecting Components
[TODO]
- Always use a state selector function that returns the minimal state subset you need.
- Expose the connected component as default export.
- Expose the unconnected component as named export for unit testing.

### Organizing Redux State

[TODO]

### Naming Things

- **action types** - use constants, *[NOUN]_\[VERB]*, eg. `USER_FETCH` or `MATERIAL_REQUEST_UPDATE`
- **action creators** - *[verb]\[Noun]()* eg. `fetchCurrentUser()`, `acceptMaterialRequest()`
- **selectors** - *get\[Noun]()* or *select\[Noun]()* eg. `getSelectedCards()`, `selectActiveRows()`


## Utility/Helper Functions
You'll likely need or want to build utility functions or libraries of them that can be reused by components and redux actions.  Organize them under a `src/utils/` folder and follow some basic guidelines.
- Group related util functions under a common name. ie, async helper functions might be in `src/utils/async.js`
- Expose each function as named export.
- Don't use a default export.
- Util functions must be pure.
- Util functions should be reusable, but have a single purpose.

> Written with [StackEdit](https://stackedit.io/).
