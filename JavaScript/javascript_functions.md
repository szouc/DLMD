# 1. **6 ways to declare JS functions**

[**original link**](https://dmitripavlutin.com/6-ways-to-declare-javascript-functions/)

<!-- TOC -->

- [1. **6 ways to declare JS functions**](#1-6-ways-to-declare-js-functions)
  - [1.1. **Function declaration**](#11-function-declaration)
  - [1.2. **Function expression**](#12-function-expression)
    - [1.2.1. **Named function expression**](#121-named-function-expression)
    - [1.2.2. **Favor named function expression**](#122-favor-named-function-expression)
  - [1.3. **Shorthand method definition**](#13-shorthand-method-definition)
  - [1.4. **Arrow function**](#14-arrow-function)
    - [1.4.1. **Context transparency**](#141-context-transparency)
    - [1.4.2. **Short callbacks**](#142-short-callbacks)
  - [1.5. **Generator function**](#15-generator-function)
  - [1.6. **One more thing:new Function**](#16-one-more-thingnew-function)

<!-- /TOC -->

- **Function declaration**
- **Function expression**
- **Shorthand method definition**
- **Arrow function**
- **Generator function**
- **Function constructor**

> Important is how the function interacts with the external components(the outer scope, the enclosing context, object that owns the method, etc) and the invocation type (regular function invocation, method invocation, constructor call, etc)

## 1.1. **Function declaration**

```js
// function declaration
function isEven(num) {  
  return num % 2 === 0;
}
isEven(24); // => true  
isEven(11); // => false  
```

```js
// Hoisted variable
console.log(hello('Aliens')); // => 'Hello Aliens!'  
// Named function
console.log(hello.name)       // => 'hello'  
// Variable holds the function object
console.log(typeof hello);    // => 'function'  
function hello(name) {  
  return `Hello ${name}!`;
}
```

## 1.2. **Function expression**

```js
var count = function(array) { // Function expression  
  return array.length;
}
var methods = {  
  numbers: [1, 5, 8],
  sum: function() { // Function expression
    return this.numbers.reduce(function(acc, num) { // func. expression
      return acc + num;
    });
  }
}
count([5, 7, 8]); // => 3  
methods.sum();    // => 14  
```

> - Assigned to a variable as an object **count = function(...) {...}**
> - Create a method on an object **sum: function() i{...}**
> - Use the function as a callback **.reduce(function(...) {...})**

### 1.2.1. **Named function expression**

```js
var getType = function funName(variable) {  
  console.log(typeof funName === 'function'); // => true
  return typeof variable;
}
console.log(getType(3));                    // => 'number'  
console.log(getType.name);                  // => 'funName'  
console.log(typeof funName === 'function'); // => false  
```

> **function funName(variable) {...}** is a named function expression. The variable **funName** is accessible within function scope, but not outside. 

### 1.2.2. **Favor named function expression**

- The error messages and call stacks show more detailed information when use the function names
- More comfortable debugging by reducing the number of anonoymous stack names
- The function name helps to understand quickly what it does
- It is possible to access the function by name inside its scope for recursive calls or detaching event listeners

## 1.3. **Shorthand method definition**

```js
var collection = {  
  items: [],
  add(...items) {
    this.items.push(...items);
  },
  get(index) {
    return this.items[index];
  }
};
collection.add('C', 'Java', 'PHP');  
collection.get(1) // => 'Java'  
```

> Notice that class syntax requires method declarations in a short form:

```js
class Star {  
  constructor(name) {
    this.name = name;
  }
  getMessage(message) {
    return this.name + message;
  }
}
var sun = new Star('Sun');  
sun.getMessage(' is shining') // => 'Sun is shining'  
```

## 1.4. **Arrow function**

```js
var absValue = (number) => {  
  if (number < 0) {
    return -number;
  }
  return number;
}
absValue(-10); // => 10  
absValue(5);   // => 5  
```

> - The arrow function does not create its own execution context, but takes it lexically (contrary to function expression or function declaration, which create own **this** depending on invocation)
> - The arrow function is anonymous: **name** is an empty string (contrary to function declaration which have a name)
> - **arguments** object is not available in the arrow function (contrary to other declaration types that provide **arguments** object)

### 1.4.1. **Context transparency**

```js
class Numbers {  
  constructor(array) {
    this.array = array;
  }
  addNumber(number) {
    if (number !== undefined) {
       this.array.push(number);
    } 
    return (number) => { 
      console.log(this === numbersObject); // => true
      this.array.push(number);
    };
  }
}
var numbersObject = new Numbers([]);  
numbersObject.addNumber(1);  
var addMethod = numbersObject.addNumber();  
addMethod(5);  
console.log(numbersObject.array); // => [1, 5]  
```

### 1.4.2. **Short callbacks**

```js
var numbers = [1, 5, 10, 0];  
numbers.some(item => item === 0); // => true  
```

## 1.5. **Generator function**

```js
function* indexGenerator(){  
  var index = 0;
  while(true) {
    yield index++;
  }
}
var g = indexGenerator();  
console.log(g.next().value); // => 0  
console.log(g.next().value); // => 1  
```

```js
var obj = {  
  *indexGenerator() { // Shorthand method
    var index = 0;
    while(true) {
      yield index++;
    }
  }
}
var g = obj.indexGenerator();  
console.log(g.next().value); // => 0  
console.log(g.next().value); // => 1  
```

## 1.6. **One more thing:new Function**

```js
function sum1(a, b) {  
  return a + b;
}
var sum2 = function(a, b) {  
  return a + b;
}
var sum3 = (a, b) => a + b;  
console.log(typeof sum1 === 'function'); // => true  
console.log(typeof sum2 === 'function'); // => true  
console.log(typeof sum3 === 'function'); // => true  
```