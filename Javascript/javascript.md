# JavaScript Notes

## The Development of JavaScript

Javascript was created by Brendan Eich in 1995 over the course of 10 days. Its purpose was to make HTML more interactive and bridge the gap between client-side Java programs and the DOM. 

The language specification is defined by the ECMA organization, hence its official name: ECMAScript. The evolution of Javascript is directed by a group of major software companies. New versions of Javascript are released yearly, and include any new features which have passed the vetting process defined by ECMA.

Due to its origins and initial development, Javascript has come to inherit a number of quirky design choices/features. In the interest of maintaining backwards compatibility, none of the oddities of Javascript have been removed/fixed. Instead, better features are offered as alternatives, and any changes to existing features must be opted into (eg. strict mode). 

## Syntax

Statements in Javascript should end in `;`. If omitted, the interpreter will usually append them where required. Relying on this behavior is not recommended.

Comment syntax is similar to Java
```js
// Single line comment

/* 
  Multi
  Line
  Comment
*/
```

Javascript operators are similar to ones found in other languages, with a few exceptions.

There are two types of equality:
```js
42 == "42" // true - "Loose equals" will coerce values to a common type before comparing them
42 === "42" // false - "Strict equals" will compare values without coercion
42 >= "42" // true - all inequality operators will coerce their operands
```

Logical operators like && and || do not necessarily evaluate to booleans. `expr1 && expr2` returns expr1 if it can be converted to false; otherwise, returns expr2. `expr1 || expr2` returns expr1 if it can be converted to true; otherwise, returns expr2.

```js
true && true;     // t && t returns true
true && false;    // t && f returns false
false && true;     // f && t returns false
false && (3 == 4); // f && f returns false
'Cat' && 'Dog';    // t && t returns Dog
false && 'Cat';    // f && t returns false
'Cat' && false;    // t && f returns false

 true || true;     // t || t returns true
false || true;     // f || t returns true
 true || false;    // t || f returns true
false || (3 == 4); // f || f returns false
'Cat' || 'Dog';    // t || t returns Cat
false || 'Cat';    // f || t returns Cat
'Cat' || false;    // t || f returns Cat
```
A full listing of operators can be found [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators)


## Types

Javascript does not support static typing; however, all values can be categorized within the following types:
* string
* number
* boolean
* null
* undefined
* object
* symbol

The `typeof` operator can be used to determine the type of a value:

```js
var a;
typeof a;				// "undefined"

a = "hello world";
typeof a;				// "string"

a = 42;
typeof a;				// "number"

a = true;
typeof a;				// "boolean"

a = null;
typeof a;				// "object"

a = undefined;
typeof a;				// "undefined"

a = { b: "c" };
typeof a;				// "object"
```

Note that `typeof null` is `object` which is an old bug that has stuck with the language from its inception. It is unlikely to be changed out of concern for backward compatibility.

## Input/Output

The console object is commonly used for outputting program information
```js
console.log("This text will be output to the console")
```

Another option within a browser is the alert function, which will display information within a popup window
```js
alert("This text is displayed in a popup")
```

User input can be received using the prompt function
```js
age = prompt("What is your age?")
```

# Sources

[You don't know JS](https://github.com/getify/You-Dont-Know-JS)
