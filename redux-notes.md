# Redux Notes

Redux is a library for managing state within an application. It's comprised of three main components: the state tree, actions, and reducers. Together, these features allow for more predictable and accessible state throughout an application. Redux acts as a single point of reference for state, which simplifies issues with synchronization of data, and facilitates a clear separation between UI and data.

## Metaphors

In Redux, the state of an application is held in a "store." The store is responsible for managing how data is accessed, altered, and propagated. To accomplish these tasks, three methods are provided by the store:

* `getState()` - Returns the current state of the store
* `dispatch(action)` - Updates the state depending on the input action object
* `subscribe(callback)` - Registers a callback function which will be called whenever the state has changed

### The State Tree

The state returned by `getState()` comes in the form of a "state tree." It is simply a large Javascript object where each property corresponds to some subsection of the state. For example:

```js
{
  recipes: [
    { … },
    { … },
    { … }
  ],
  ingredients: [
    { … },
    { … },
    { … },
    { … },
    { … },
    { … }
  ],
  products: [
    { … },
    { … },
    { … },
    { … }
  ]
}
```

### Actions

To update the state, the store will accept objects called "actions." Actions define the various ways an application may interact with the state. These are application defined and must follow a specific structure. All action objects must have a `type` property, which will be used to dispatch the action to the correct handler(s). Typically, an action object will contain additional properties which will be used in generating the new state. It is good practice to limit the additional information in the object to the bare minimum needed to accomplish the action. Here are some examples:

```js
{
  type: 'ADD_RECIPE',
  recipe: {
    id: 3,
    name: 'Pancakes',
    ingredients: ['2 Eggs', '500mL Milk', ...],
    steps: ['1. Whisk eggs and milk in a bowl', '2. ...', ...]
  }
}

{
  type: 'REMOVE_RECIPE',
  id: 3 // keep payload information to a minimum - passing the whole recipe object would be unnecessary!
}
```

### Action Creators

Many parts of an application will want to perform actions on the state. Creating an action object from scratch each time is repetitive and error prone. It is helpful then, to define functions called "action creators" which are passed the relevant payload information, and create the appropriate action object. 

```js
const REMOVE_RECIPE = 'REMOVE_RECIPE';
const ADD_RECIPE = 'ADD_RECIPE';

function removeRecipe(id) {
  return {
    type: REMOVE_RECIPE,
    id: id
  }
}

function addRecipe(recipe) {
  return {
    type: ADD_RECIPE,
    recipe
  }
}
```

Constant definitions like `REMOVE_RECIPE = 'REMOVE_RECIPE'` are useful for a number of reasons. For one, they help document all of the different actions available on the store. Another reason is that typing strings directly is prone to errors. A typo in a string literal will not be caught by the interpreter, but referencing an undefined variable will. 

### Reducers

Once an application has defined its actions, they can be passed to the store through the `dispatch(action)` method. This method will dispatch the action to a reducer. A reducer is a pure function which receives the state and an action, and returns the new state. For that reason, it is said to "reduce" a state and an action into a new state. A pure function must satisfy three criteria:

* Always return the same result given the same arguments
* Depend solely on the arguments passed to it (i.e. no reliance on outside state)
* Do not produce side effects such as API requests or I/O operations

Having reducers be pure functions ensures that changes to state are predictable and easy to reason about. A reducer is provided by an application when the store is first created. From there redux will handle calling it with any actions it receives. Here's an example of a simple reducer:

```js
function recipes(state = [], action) { // the default state = [] assignment handles the case when no state has been initialized yet
  switch(action.type) {
    case ADD_RECIPE:
      return state.concat([action.recipe]);
    case REMOVE_RECIPE:
      return state.filter((recipe) => (recipe.id != action.id));
    default:
      return state;
  }
}
```

#### Combining Reducers

Oftentimes the state of an application is split into a number of different parts. For example, a twitter application may want to maintain information about their users, settings, and tweets. This might look something like this:

```js
{
  users: {},
  settings: {},
  tweets: {
    btyxlj: {
      id: 'btyxlj',
      text: 'What is a jQuery?',
      author: {
        name: 'Joe',
        id: '@joe',
        avatar: 'twt.com/joe.png'
      }  
    }
  }
}
```

It can become cumbersome and confusing to have one reducer handle all of the actions possible on the different properties of the state tree. Instead, it is useful to define a single reducer for each slice of the state, then combine them into a single reducer which can be passed into redux. For the state tree above, we could have reducers for each of the top level `users` and `settings` properties:

```js
function users(state = {}, action) {
  switch (action.type) {
    case ADD_USER :
      ...
    case REMOVE_USER :
      ...
    default
      return state
  }
}

function settings(state = {}, action) {
  switch (action.type) {
    case CHANGE_SETTING :
      ...
    default
      return state
  }
}
```

These can be combined into something referred to as a "root reducer" which can return the entire state tree while delegating each part to the appropriate reducer:

```js
function rootReducer(state = {}, action) {
  return {
    users: users(state.users, action),
    settings: settings(state.settings, action)
  }
}
```

Since this pattern is so common with Redux, a dedicated function is provided for combining reducers in this way:

```js
const rootReducer = Redux.combineReducers({
  users,  // The function automatically creates a state property for users, and passes the appropriate arguments into the users reducer
  settings
})
```

Lets take a closer look at the tweets property of the state:

```js
tweets: {
  btyxlj: {
    id: 'btyxlj',
    text: 'What is a jQuery?',
    author: {
      name: 'Joe',
      id: '@joe',
      avatar: 'twt.com/joe.png'
    }  
  }
}
```

Because there are multiple levels of properties for tweets, having a single reducer recreate the entire object can quickly get out of hand when making changes to deeply nested properties:

```js
function tweets(state = {}, action) {
  switch(action.type) {
    case ADD_TWEET :
      return {
        ...state,
        [action.tweet.id] : action.tweet
      } // Since reducers are pure functions we must return a completely new state. Do not alter the existing state directly!
    case REMOVE_TWEET :
      ...
    case UPDATE_AVATAR :
      return { // So many spread operators @_@
        ...state,
        [action.tweet.id]: {
          ...state[action.tweet.id],
          author: {
            ...state[action.tweet.id].author,
            avatar: action.newAvatar // The only significant change in this whole case
          }
        }
      }
    default :
      return state;
  }
}
```

Just as we delegated the top level state properties to their own reducers, it makes sense to do the same for the tweets state. In general, whenever a property refers to another object, it is useful to define a new reducer to handle actions on that object. Taking a look at the tweets property, we can break it down into three reducers:

```js
// handles actions on the entire tweets object
function tweets(state = {}, action) {
  switch(action.type) {
    case ADD_TWEET :
      ...
    case REMOVE_TWEET :
      ...
    case EDIT_TWEET :
      return {
        ...state,
        [action.tweetId]: tweet(state[action.tweetId], action) // delegate to the tweet reducer
      }
    case UPDATE_AVATAR :
      return {
        ...state,
        [action.tweetId]: tweet(state[action.tweetId], action) // delegate to the tweet reducer
      }
    default :
      state
  }
}

// handles actions for individual tweet objects
function tweet(state = {}, action) {
  switch (action.type) {
    case EDIT_TWEET :
      return {
        ...state,
        text: action.text
      }
    case UPDATE_AVATAR :
      return {
        ...state,
        author: author(state.author, action) // delegate to the author reducer
      }
    default :
      return state
  }
}

// handles actions for author objects
function author(state = {}, action) {
  switch (action.type) {
    case : UPDATE_AVATAR
      return {
        ...state,
        avatar: action.newAvatar
      }
    default :
      state
  }
}
```

## A Simple Implementation of a Redux Store Application

The following is a bare bones implementation of a redux store with the necessary `getState()`, `dispatch(action)`, and `subscribe(listener)` methods:

```js
function createStore(reducer) { // an application-defined reducer is passed in
  let state // state tree is referenced here
  let listeners = [] // array of listeners currently subscribed for updates

  const getState = () => state

  const subscribe = (listener) => {
    listeners.push(listener)
    return () => {
      listeners = listeners.filter((l) => l !== listener)
    } // subscribe adds the listener, then returns a function which can be called to remove the listener
  }

  const dispatch = (action) => {
    state = reducer(state, action) // calls the reducer to get the new state
    listeners.forEach((listener) => listener()) // since the state has changed, all listeners are called
  }

  return {
    getState,
    subscribe,
    dispatch
  }
}
```

A simple application making use of the store may look like this:

```js
// Actions
const ADD_RECIPE = 'ADD_RECIPE';

// Action Creators
function addRecipe(recipe) {
  return {
    type: ADD_RECIPE,
    recipe
  }
}

// Reducers
function recipes(state = [], action) {
  switch(action.type) {
    case ADD_RECIPE:
      return state.concat([action.recipe]);
    default:
      return state;
  }
}

// Interacting with the store
const store = createStore(recipes);

store.subscribe(() => {
  console.log('The new state is: ', store.getState())
});

store.dispatch(addRecipe({
  id: 3,
  name: 'Pancakes',
  ingredients: ['2 Eggs', '500mL Milk'],
  steps: ['1. Whisk eggs and milk in a bowl', '2. ...']
}));
```

## Middleware

When an action is fired, the default behavior of Redux is to dispatch the action to a reducer, which will then immediately generate the new state. However, there are times when an application may want to do some computation in between the action getting dispatched and the new state being created. Any code that runs between these two points in time is called middleware. Some common middleware tasks are:

* Producing a side effect (e.g., logging information about the store)
* Processing the action itself (e.g., making an asynchronous HTTP request)
* Redirecting the action (e.g., to another piece of middleware)
* Dispatching supplementary actions

The way middleware functions are declared can seem a little strange because of what Redux expects to receive:

```js
function middleware (store) {
  return function (next) {
    return function (action) {
      // middleware code
    }
  }
}
```

So a middleware function is a higher-order function (a function which accepts/returns other functions) which is eventually called with the store, the next middleware function (or the dispatch function), and action. In order to ensure control flows through each middleware function to dispatch, make sure each function returns the value of calling `next(action)`. Here's an example of a simple logging middleware:

```js
const logger = (store) => (next) => (action) => {
  console.group()
  console.log('The action is ', action)
  result = next(action)
  console.log('The result is ', result)
  console.groupEnd()

  return result
}
```

## Using Redux

Because of the flow Redux imposes on interacting with data, a number of architectural questions must be answered before implementing an application with Redux. How will actions be dispatched? How will the store's state be shared? How will updates to the store be propagated? How should reducers, actions, action creators, and components be organized within an application? The following will present increasingly sophisticated implementations of a todo/goals application using Redux.

### Redux with Javascript and HTML

The UI is fairly straightforward:

```html
<div>
  <h1>Todo List</h1>
  <input id="todo" type="text" placeholder="Add Todo" />
  <button id="todoBtn">Add Todo</button>
  <ul id="todos"></ul>
</div>
<div>
    <h1>Goals</h1>
    <input id="goal" type="text" placeholder="Add Goal" />
    <button id="goalBtn">Add Todo</button>
    <ul id="goals"></ul>
</div>
```

Redux specific code like actions, reducers, and middleware are defined globally alongside application code:

```js
// Action Types
const ADD_TODO = 'ADD_TODO'
const REMOVE_TODO = 'REMOVE_TODO'
const TOGGLE_TODO = 'TOGGLE_TODO'
const ADD_GOAL = 'ADD_GOAL'
const REMOVE_GOAL = 'REMOVE_GOAL'

// Action creators
function addTodoAction (todo) {
  return {
    type: ADD_TODO,
    todo,
  }
}

function removeTodoAction (id) {
  return {
    type: REMOVE_TODO,
    id,
  }
}

function toggleTodoAction (id) {
  return {
    type: TOGGLE_TODO,
    id,
  }
}

function addGoalAction (goal) {
  return {
    type: ADD_GOAL,
    goal,
  }
}

function removeGoalAction (id) {
  return {
    type: REMOVE_GOAL,
    id,
  }
}

// Reducers
function todos (state = [], action) {
  switch(action.type) {
    case ADD_TODO:
      return state.concat([action.todo])
    case REMOVE_TODO:
      return state.filter((todo) => (todo.id !== action.id))
    case TOGGLE_TODO:
      return state.map((todo) => todo.id !== action.id ? todo : 
        Object.assign({}, todo, { complete: !todo.complete }))
    default:
      return state
  }
}

function goals (state = [], action) {
  switch(action.type) {
    case ADD_GOAL:
      return state.concat([action.goal])
    case REMOVE_GOAL:
      return state.filter((goal) => goal.id !== action.id)
    default:
      return state
  }
}


// Middleware
const logger = (store) => (next) => (action) => {
  console.group()
  console.log('The action is ', action)
  result = next(action)
  console.log('The result is ', result)
  console.groupEnd()

  return result
}

// Instantiating the store
const store = Redux.createStore(Redux.combineReducers({
  todos,
  goals,
}), Redux.applyMiddleware(logger))
```

To update the DOM with the state managed by Redux, we can re-render the list of todos/goals each time the state changes:

```js
store.subscribe(() => {
  const { goals, todos } = store.getState()

  document.getElementById('todos').innerHTML = ''
  document.getElementById('goals').innerHTML = ''

  todos.forEach(addTodoToDOM)
  goals.forEach(addGoalToDOM)
})
```

Actions are dispatched by elements which the user will interact with. For instance, the 'Add Todo' button is assigned a click handler which dispatches the add todo action. Similarly, other interactive UI elements are assigned handlers for their respective actions:

```js
// Configuring Todo/Goal list elements
function addTodoToDOM (todo) {
  const node = document.createElement('li')
  const text = document.createTextNode(todo.name)

  const removeBtn = createRemoveButton(() => {
    store.dispatch(removeTodoAction(todo.id))
  })

  if (todo.complete) node.style.textDecoration = 'line-through'
  node.appendChild(text)
  node.append(removeBtn)
  node.addEventListener('click',
    () => {store.dispatch(toggleTodoAction(todo.id))})

  document.getElementById('todos').appendChild(node)
}

function addGoalToDOM (goal) {
  const node = document.createElement('li')
  const text = document.createTextNode(goal.name)
  const removeBtn = createRemoveButton(() => {
    store.dispatch(removeGoalAction(goal.id))
  })

  node.appendChild(text)
  node.append(removeBtn)

  document.getElementById('goals').appendChild(node)
}

document.getElementById('todoBtn')
  .addEventListener('click', addTodo)
  
document.getElementById('goalBtn')
  .addEventListener('click', addGoal)

function createRemoveButton (onClick) {
  const removeBtn = document.createElement('button')
  removeBtn.innerHTML = 'X'
  removeBtn.addEventListener('click', onClick)
  return removeBtn
}

// Handlers
function addTodo () {
  const input = document.getElementById('todo')
  const name = input.value
  input.value = ''

  store.dispatch(addTodoAction({
    name,
    complete: false,
    id: generateId()
  }))
}

function addGoal () {
  const input = document.getElementById('goal')
  const name = input.value
  input.value = ''

  store.dispatch(addGoalAction({
    name,
    id: generateId()
  }))
}
```

In this example, integration with Redux was relatively straightforward. The app is organized as one monolithic file, which greatly simplifies scoping. As a result, interactions with state and actions can be undertaken from anywhere in the application. Action dispatches are handled by click handlers, and the dynamic content is re-rendered entirely for every change in state, which is generally quite inefficient. Overall, this example is a basic demonstration of how Redux can be used in an application, but does not serve as a good example for larger projects. Using React would greatly increase the modularity of our components while improving rendering performance.

### Redux with React

Implementing the same todo/goals app in React greatly improves modularity, but does require a little more code to manage access to the state. The store is passed along as a prop down the component hierarchy, starting with the main app component. Because the store is not part of any component's state, React will not automatically handle re-rendering on changes to this state (since `setState()` will never be called). As a result, the main app component subscribes a listener to the store which will call `forceUpdate()` to trigger a re-render.

```js
class App extends React.Component {
  componentDidMount () {
    const { store } = this.props

    store.subscribe(() => this.forceUpdate())
  }

  render() {
    const { store } = this.props
    const { todos, goals } = store.getState()

    return (
      <div>
        <Todos todos={todos} store={this.props.store}/>
        <Goals goals={goals} store={this.props.store}/>
      </div>
    )
  }
}

ReactDOM.render(
  <App store={store}/>,
  document.getElementById('app')
)

class Todos extends React.Component {
  addItem = (e) => {
    e.preventDefault()
    const name = this.input.value
    this.input.value = ''
    
    this.props.store.dispatch(addTodoAction({
      name,
      complete: false,
      id: generateId()
    }))
  }

  removeItem = (todo) => {
    this.props.store.dispatch(removeTodoAction(todo.id))
  }

  toggleItem = (id) => {
    this.props.store.dispatch(toggleTodoAction(id))
  }

  render() {
    return (
      <div>
        <h1>Todo List</h1>
        <input
          type='text'
          placeholder='Add Todo'
          ref={(input) => this.input = input} 
          // a ref here is not best practice, but makes sharing the input value quick and easy
        />
        <button onClick={this.addItem}>Add Todo</button>
        <List
          items={this.props.todos}
          remove={this.removeItem}
          toggle={this.toggleItem}
        />
      </div>
    )
  }
}

class Goals extends React.Component {
  addItem = (e) => {
    e.preventDefault()
    const name = this.input.value
    this.input.value = ''

    this.props.store.dispatch(addGoalAction({
      name,
      id: generateId()
    }))
  }

  removeItem = (goal) => {
    this.props.store.dispatch(removeGoalAction(goal.id))
  }

  render() {
    return (
      <div>
        <h1>Goals</h1>
        <input
          type='text'
          placeholder='Add Goal'
          ref={(input) => this.input = input}
        />

        <button onClick={this.addItem}>Add Goal</button>
        <List items={this.props.goals} remove={this.removeItem} />
      </div>
    )
  }
}

function List (props) {
  return (
    <ul>
      {props.items.map((item) => (
        <li key={item.id}>
          <span
            onClick={() => props.toggle && props.toggle(item.id)}
            style={{textDecoration: item.complete ? 'line-through' : 'none'}}
          >
            {item.name}
          </span>
          <button onClick={() => props.remove(item)}>X</button>
        </li>
      ))}
    </ul>
  )
}
```

This iteration of the todos/goals app has divided the monolithic Javascript version into smaller components. To keep things simple, store state is passed along as a prop starting with the root app component, however, in a real world application this data would be fetched asynchronously. Luckily, there exist a few common patterns in Redux which will help to manage asynchronous requests.