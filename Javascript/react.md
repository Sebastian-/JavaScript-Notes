# React Notes

## Using JSX

JSX is a syntax extension for JavaScript which makes declaring and using React elements simpler. It looks very similar to HTML, but actually compiles down to a JavaScript object at the end of the day. 

```jsx
  const element = <h1>Hello, world!</h1>;
```

Inherent to using JSX is the fact that JavaScript logic and markup is combined in a single place. This differs from other approaches, where logic and markup are in separate files. Instead of separating projects into layers/technologies, React encourages separating projects into loosely coupled components which do everything required to handle a specific concern. Curly braces in JSX can contain any valid Javascript expression.

```jsx
const element = <h1>Hello, {foo}</h1>;

const element = (
  <h1>
    Hello, {foo(bar)}!
  </h1>
);
```

Additionally, React DOM will escape anything embedded in JSX, making it safe to embed user input. Everything is converted to a string before rendering.

```jsx
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

Like HTML, JSX elements can have attributes. Usually they mirror the same ones available in HTML, except their names are formatted in camelCase, rather than lower case.

```jsx
const element = <div tabIndex="0"></div>;

const element = <img src={user.avatarUrl}></img>;
```

JSX elements may contain child elements. If empty, the closing tag can be omitted by adding a `/` to the opening tag. It is recommended to wrap multi-line JSX with parentheses to avoid errors due to automatic semicolon insertion.

```jsx
const element = <img src={user.avatarUrl} />;

const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

Babel compiles JSX into React.createElement() calls, which go on to return a JavaScript object representing the element.

```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);

const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);

// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

## Rendering Elements

React elements are the smallest building blocks of React apps. They are used to represent the user interface of the app at any given moment. All react elements are rendered as children of a root DOM node. The element to be rendered, alongside this root node, are passed to ReactDOM.render().

```jsx
// Somewhere in your html
<div id="root"></div>

// React code
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

Despite being regular JavaScript objects under the hood, elements are not mutated directly. Instead, a brand new element is created to represent the new state of the app. ReactDOM will compare the differences between the new and current element, and only update the UI where it needs to change. This allows for developers to focus on how the UI should look at a given time, rather than how to transition from one state to another.

```jsx
// Despite this function calling ReactDOM with a new element each tick, 
// only the changing time will be updated in the DOM. All other DOM 
// elements will remain unchanged.

function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

## Components and Props

Components are essentially functions which accept an object of properties, and return some React elements in response. They are meant to define reuseable and independent pieces of an interface. Components can be defined either as functions, or as ES6 classes which implement a `render()` function. Function definitions are generally more efficient and easier to understand, while class definitions have additional API features available to them. Finally, component names should always start with a capital letter! React will assume lower case components are DOM elements and compile their JSX differently (e.g. `<div />` compiles to `React.createElement('div')` while `<Foo />` compiles to `React.createElement(Foo)`).

```jsx
// Defining components
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// Rendering components
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

Components should be pure functions with respect to their props. This means components passed the same props should return the same elements, and components should not alter their props in any way. Instead, a component can declare and manage state to handle dynamically changing variables.
