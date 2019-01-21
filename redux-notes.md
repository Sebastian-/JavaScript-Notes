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

So a middleware function is a higher-order function (a function which accepts/returns other functions) which eventually receives the store, the next function in the chain on the way to the reducer, and an action. In order to ensure control flows through each middleware function, make sure each function returns the value of calling `next(action)`. Here's an example of a simple logging middleware:

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

### Asynchronous Requests with Redux (Redux Thunk)

Typically, a web application will persist its state on a remote server and interact with it as required. While Redux can easily manage client-side state, some extra implementation work is required to make use of server-side state. 

To kick things off, an application will usually fetch some initial set of data to display to the user, like an initial list of todo/goals. It would be inefficient to dispatch an action to add each item individually, so it makes sense to define a load action which will add all the items at once.

```js
function receiveDataAction (todos, goals) {
  return {
    type: RECEIVE_DATA,
    todos,
    goals,
  }
}
```

Handling this action in the reducers is relatively straightforward:

```js
function todos (state = [], action) {
  switch(action.type) {
    // ...
    case RECEIVE_DATA:
      return action.todos
    // ...
  }
}

function goals (state = [], action) {
  switch(action.type) {
    // ...
    case RECEIVE_DATA:
      return action.goals
    // ...
  }
}
```

Now comes the question of where this action should be dispatched. A natural choice is to fetch the data right as the app component is mounted:

```js
class App extends React.Component {
  componentDidMount () {
    const { store } = this.props
    store.subscribe(() => this.forceUpdate())

    Promise.all([
      API.fetchGoals(),
      API.fetchTodos()
    ]).then(([ goals, todos ]) => {
      store.dispatch(receiveDataAction(todos, goals))
    })
  }
  
  // ...

}
```

This works, but will result in an awkward re-render once the initial data is loaded. A more user-friendly approach would be to display some sort of loading screen while the information is still being gathered. To implement this, a `loading` flag can be added to the store in order to conditionally render some loading UI.

```js
// Reducer to handle loading state - loading is true until the RECEIVE_DATA action is dispatched
function loading (state = true, action) {
  switch(action.type) {
    case RECEIVE_DATA:
      return false
    default:
      return state
  }
}

// App component conditionally renders based on loading state
render() {
  const { store } = this.props
  const { todos, goals, loading } = store.getState()

  if (loading === true) {
    return <h3>Loading</h3>
  }

  return (
    <div>
      <Todos todos={todos} store={this.props.store}/>
      <Goals goals={goals} store={this.props.store}/>
    </div>
  )
}
```

Now that the application has some initial state, any changes to this state must be transmitted to the server in order to keep them in sync. A simple approach here is to make API calls within components alongside action dispatches. 

```js
class Todos extends React.Component {
  addItem = (e) => {
    e.preventDefault()

    return API.saveTodo(this.input.value)
      .then((todo) => {
        this.props.store.dispatch(addTodoAction(todo))
        this.input.value = ''
      })
      .catch(() => {
        alert('There was an error adding a todo. Try again.')
      })
  }

  removeItem = (todo) => {
    this.props.store.dispatch(removeTodoAction(todo.id))
    
    return API.deleteTodo(todo.id)
      .catch(() => {
        this.props.store.dispatch(addTodoAction(todo))
        alert(`An error occurred while removing ${todo.name}. Try again.`)
      })
  }

  toggleItem = (id) => {
    this.props.store.dispatch(toggleTodoAction(id))

    return API.saveTodoToggle(id)
      .catch(() => {
        this.props.store.dispatch(toggleTodoAction(id))
        alert('An error occurred while toggling. Try again.')
      })
  }

  // ...

}
```

Notice how `removeItem` and `toggleItem` dispatch an action before receiving confirmation from the server that the request succeeded. This is known as "Optimistic Updating" because the UI will update under the assumption that the request will succeed. This makes updates from the user's perspective seem instantaneous, despite the fact that the API request will take some time to resolve. If the API request fails, the client-side state can simply be reverted. In the case of `addItem`, making an optimistic update is not trivial because a new todo/goal item will need to have an id assigned to it by the server.

The end goal of a React component is to render some UI based on the state of an application. After all, the `render` method is the only required method for a React component. Managing API calls within components can greatly complicate and obscure this primary purpose, so it would be nice if they could be handled somewhere else. Ideally, a component could simply dispatch an action and have all asynchronous work be performed somewhere along the dispatch path. If this work can be delegated to an action creator, then a component can simply dispatch that action as before:

```js
removeItem = (todo) => {
  this.props.store.dispatch(handleDeleteTodo(todo))
}
```

Action creators are supposed to return action objects. In this case however, the action to be dispatched changes depending on the result of some asynchronous work (`removeTodoAction` is dispatched optimistically, but if the API call fails, the removal must be reverted with an `addTodoAction`). To handle this scenario, a common pattern is to return a function which encapsulates the asynchronous work. Then, some middleware can be responsible for invoking this function before it reaches the reducers/other middleware. This type of middleware is called a "Thunk."

```js
// Action creator returns a function instead of an action object
function handleDeleteTodo (todo) {
  return (dispatch) => {
    dispatch(removeTodoAction(todo.id))

    return API.deleteTodo(todo.id)
      .catch(() => {
        dispatch(addTodoAction(todo))
        alert(`An error occurred while removing ${todo.name}`)
    })
  }
}

// Thunk middleware
const thunk = (store) => (next) => (action) => {
  if (typeof action === 'function') {
    return action(store.dispatch)
  }

  return next(action)
}
```

Redux will apply middleware in the order it is declared when the store is created. Therefore, it's important to place the thunk middleware first, so that it can intercept any functional actions before they are passed to middleware which cannot handle them. Using functional actions and thunk middleware is a common pattern, so a library called `ReduxThunk` is available for use in applications which need it.

```js
const store = Redux.createStore(
  Redux.combineReducers({todos, goals, loading,}), 
  Redux.applyMiddleware(ReduxThunk.default, checker, logger)
)
```

### React-Redux Bindings

Since Redux is not specifically tailored for use in React, there are some improvements that can be made to their interface. For one, it would be nice if the store did not have to be passed down as a prop through nested components. Doing so can become very tedious, and some components may not even use the store as it is passed along. Subscribing and unsubscribing can also be repetitive, and needlessly complicates components. React-Redux addresses these issues by providing a simple API to cleanly and easily connect components to the store. A simple implementation of the library is provided below, which matches the API used by the official version.

Passing state down from a root component to a lower level component can become very tedious and inefficient. To address this, React allows for state to be accessed from anywhere in a component tree through its Context API. To make state available, wrap the root of the component hierarchy with `<Context.Provider value={state}>`, passing in any state to a prop called value. Then, child components can access the state by rendering from within `<Context.Consumer>`.

```js
const Context = React.createContext()

// A simple wrapper for Context.Provider to make passing the store cleaner
class Provider extends React.Component {
  render () {
    return (
      <Context.Provider value={this.props.store}>
        {this.props.children} 
      </Context.Provider>
    )
  }
}

ReactDOM.render(
  <Provider store={store}>
    <ConnectedApp />
  </Provider>,
  document.getElementById('app')
) // Without the Provider wrapper, the wrapping tag would be <Context.Provider value={store}>

class ConnectedApp extends React.Component {
  render() {
    return (
      <Context.Consumer>
        {(store) => (
          <App store={store} />
        )}
      </Context.Consumer>
    )
  }
}

class App extends React.Component {
  componentDidMount () {
    const { store } = this.props
    store.subscribe(() => this.forceUpdate())
    store.dispatch(handleInitialData())
  }

  render() {
    const { store } = this.props
    const { loading } = store.getState()

    if (loading === true) {
      return <h3>Loading</h3>
    }

    // ...
  }
}
```

Notice the separation of concerns between getting the store in `ConnectedApp` and using the store in `App`. This structure is commonly employed, and the component managing the store is called the "Connected Component" while its child is called the "Presentational Component." Connected components are meant to handle processing related to the store/state, while presentational components are strictly for rendering UI.

At this point, the `App` component still contains some store logic in the form of `store.subscribe(() => this.forceUpdate())`. In fact, many components interested in store state will contain this invocation. Since it is logic related to the store, it should be handled by a connected component rather than a presentational one. Additionally, connected components tend to share the same structure: they render a presentational component wrapped with `Context.Consumer` and pass in whatever props the presentational component needs. Also, any presentational component interacting with store data will need to be passed the `dispatch` function. All of this repetitive work can be encapsulated in a `connect` function: 

```js
function connect (mapStateToProps) {

  return (Component) => {

    class Receiver extends React.Component {
      componentDidMount () {
        const { subscribe } = this.props.store

        this.unsubscribe = subscribe(() => {
          this.forceUpdate()
        })
      }

      componentWillUnmount () {
        this.unsubscribe()
      }

      render () {
        const { dispatch, getState } = this.props.store
        const state = getState()
        const stateNeeded = mapStateToProps(state)

        return <Component {...stateNeeded} dispatch={dispatch} />
      }
    }

    class ConnectedComponent extends React.Component {
      render () {
        return (
          <Context.Consumer>
            {(store) => (
              <Receiver store={store}/>
            )}
          </Context.Consumer>
        )
      }
    }

    return ConnectedComponent
  }
}

/* Invoking connect
    connect(mapStateToProps)(component)
    - mapStateToProps is a function taking in the store state and returning an 
      object with the props required to render the component
    - component is simply the presentational component which will use the props
      from mapStateToProps
*/
const ConnectedApp = connect((state) => ({
  loading: state.loading
}))(App)
```

 With this, the ConnectedApp component has been reduced to a simple function call. The implementation here is simpler than the official react-redux library, but provides a good model for how the library works under the hood.
 