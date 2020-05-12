# Chapter 1 - Values, Types, and Operators

## Numbers

- Javascript numbers are 64 bit
- Range of decimal numbers: +/- 9e10<sup>18</sup>
- Range of integers: +/- 9e10<sup>15</sup>
- Special Numbers
  - `Infinity`
  - `-Infinity`
  - `NaN`

## Strings

- Strings are delimited by single quotes, double quotes, or backticks
- Special characters can be escaped using a backslash ( \\ )
  - \\n denotes a newline in quoted strings
  - \\t denotes a tab
- Javacript strings adhere to the Unicode standard
- Strings can be concatenated using the `+` operator
- Strings delimited by backticks are called 'template literals'

  - Only strings declared using backticks will respect newline formatting
  - Template literals can contain embedded javascript expressions

  ```javascript
  `half of 100 is ${100 / 2}`;
  ```

## Unary Operators

- Unary operators can be applied to a single operand
- Examples
  - `typeof` returns a string corresponding to the type of the operand
  - `-` negates a number

## Boolean Values

- Denoted by `true` and `false`
- As in other languages, represents the result of a comparison (`>, <, ==, !==` etc.)
  - All values are equal to themselves except `NaN == NaN // false`
- Unlike other languages, any value can be interpreted as a boolean. All values are 'truthy' except the following:
  - `false`
  - `0` The number 0
  - `-0` Negative 0
  - `0n` BigInt value of 0
  - `""` Empty string
  - `null`
  - `undefined`
  - `NaN`

## Logical Operators

- Unary

  - `!` returns the opposite boolean value

- Binary

  - `&&` logical AND
  - `||` logical OR

- Ternary

  - `condition ? expression-if-true : expression-if-false`

- Short-Circuiting
  - Operands in boolean expressions are only evaluated if their values are required to determine the overall result of the expression.
  ```javascript
  console.log(null || "user");
  // → user
  console.log("Agnes" || "user");
  // → Agnes
  ```

## Empty Values

- `undefined` and `null` represent meaningless values
- There is little practical difference between them

## Automatic Type Conversion

- If the types of two operands do not match, in many cases, Javascript will automatically cast one of them in order to evaluate the expression

```javascript
console.log(8 * null);
// → 0
console.log("5" - 1);
// → 4
console.log("5" + 1);
// → 51
console.log("five" * 2);
// → NaN
console.log(false == 0);
// → true
```

- This behavior can be bypassed using `===` or `!==`
