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

To update the state, the store will accept objects called "actions." Actions define the various ways an application may interact with the state. All action objects must have a `type` property, which will be used to dispatch the action to the correct handler(s). Typically, an action object will contain additional properties which will be used in generating the new state. It is good practice to limit the additional information in the object to the bare minimum needed to accomplish the action. Here are some examples:

```js
{
  type: 'ADD_RECIPE',
  recipe: {
    id: 3
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

