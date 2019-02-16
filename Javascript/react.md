# React Notes

## Using JSX

JSX is a syntax extension for JavaScript which makes declaring and using React elements simpler. It looks very similar to HTML, but actually compiles down to a JavaScript object at the end of the day. 

```jsx
  const element = <h1>Hello, world!</h1>;
```

Inherent to using JSX is the fact that JavaScript logic and markup is combined in a single place. This differs from other approaches, where logic and markup are in separate files. Instead of separating projects into layers/technologies, React encourages separating projects into loosely coupled components which handle everything required to handle a specific concern.

Curly braces in JSX can contain any valid Javascript expression.

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

It is recommended to wrap multi-line JSX with parentheses to avoid errors due to automatic semicolon insertion.

Like HTML, JSX elements can have attributes. Usually they mirror the same ones available in HTML, except their names are formatted in camelCase, rather than all lower case.

```jsx
const element = <div tabIndex="0"></div>;

const element = <img src={user.avatarUrl}></img>;
```

JSX elements may contain child elements. If empty, the closing tag can be omitted by adding a `/` to the opening tag.

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
