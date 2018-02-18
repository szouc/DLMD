**Gentle explanation of 'this' keyword in JavaScript**

[**original link**](https://dmitripavlutin.com/gentle-explanation-of-this-in-javascript/)

<!-- TOC -->

- [1. **The mystery of this**](#1-the-mystery-of-this)
- [2. **Function invocation**](#2-function-invocation)
  - [2.1. **this in function invocation**](#21-this-in-function-invocation)
  - [2.2. **this in function invocation, strict mode**](#22-this-in-function-invocation-strict-mode)
  - [2.3. **Pitfall:this in an inner function**](#23-pitfallthis-in-an-inner-function)
- [3. **Method invocation**](#3-method-invocation)
  - [3.1. **this in method invocation**](#31-this-in-method-invocation)
  - [3.2. **Pitfall:separating method from it object**](#32-pitfallseparating-method-from-it-object)
- [4. **Constructor invocation**](#4-constructor-invocation)
  - [4.1. **this in constructor invocation**](#41-this-in-constructor-invocation)
  - [4.2. **Pitfall:forgetting about new**](#42-pitfallforgetting-about-new)
- [5. **Indirect invocation**](#5-indirect-invocation)
  - [5.1. **this in indirect invocation**](#51-this-in-indirect-invocation)
- [6. **Bound function**](#6-bound-function)
  - [6.1. **this in bound function**](#61-this-in-bound-function)
  - [6.2. **Tight context binding**](#62-tight-context-binding)
- [7. **Arrow function**](#7-arrow-function)
  - [7.1. **this in arrow function**](#71-this-in-arrow-function)
  - [7.2. **Pitfall:defining method with arrow function**](#72-pitfalldefining-method-with-arrow-function)

<!-- /TOC -->

# 1. **The mystery of this**

In JavaScript the situation is different: **this** is the current execution context of a function. The language has 4 function invocation types:

* function invocation: **alert('Hello World!')**
* method invocation: **console.log('Hello World!')**
* constructor invocation: **new RegExp('\\d')**
* indirect invocation: **alert.call(undefined, 'Hello World!')**

Before starting, let's familiarize with a couple of terms:

* **Invocation** of a function is executing the code that makes the body of a function, or simply calling the function. For example **parseInt** function **invocation** is **parseInt('15')**.
* **Context** of an invocation is the value of this within function body. For example the invocation of **map.set('key', 'value')** has the context **map**.
* **Scope** of a function is the set of variables, objects, functions accessible within a function body.

# 2. **Function invocation**

```js
function hello(name) {
  return "Hello " + name + "!";
}
// Function invocation
var message = hello("World");
console.log(message); // => 'Hello World!'
```

## 2.1. **this in function invocation**

> **this** is the **global object** in a function invocation

![2-1.png](http://upload-images.jianshu.io/upload_images/5518635-9c8fd44f997541d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
function sum(a, b) {
  console.log(this === window); // => true
  this.myNumber = 20; // add 'myNumber' property to global object
  return a + b;
}
// sum() is invoked as a function
// this in sum() is a global object (window)
sum(15, 16); // => 31
window.myNumber; // => 20
```

## 2.2. **this in function invocation, strict mode**

> **this** is **undefined** in a function invocation in strict mode

![3-1.png](http://upload-images.jianshu.io/upload_images/5518635-977f0dc1cbf83f6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
function multiply(a, b) {
  "use strict"; // enable the strict mode
  console.log(this === undefined); // => true
  return a * b;
}
// multiply() function invocation with strict mode enabled
// this in multiply() is undefined
multiply(2, 5); // => 10
```

## 2.3. **Pitfall:this in an inner function**

* A common trap with the function invocation is thinking that **this** is the same in an inner function as in the outer function.
* Correctly the context of the inner function depends only on invocation, but not on the outer function's context.

```js
var numbers = {
  numberA: 5,
  numberB: 10,
  sum: function() {
    console.log(this === numbers); // => true
    function calculate() {
      // this is window or undefined in strict mode
      console.log(this === numbers); // => false
      return this.numberA + this.numberB;
    }
    return calculate();
  }
};
numbers.sum(); // => NaN or throws TypeError in strict mode
```

> To solve the problem, **calculate** function should be executed with the same context as the **sum** method, in order to access **numberA** and **numberB** properties.

# 3. **Method invocation**

Understanding the distinction between function invocation and method invocation helps identifying correctly the context.

```js
["Hello", "World"].join(", "); // method invocation
({
  ten: function() {
    return 10;
  }
}.ten()); // method invocation
var obj = {};
obj.myFunction = function() {
  return new Date().toString();
};
obj.myFunction(); // method invocation

var otherFunction = obj.myFunction;
otherFunction(); // function invocation
parseFloat("16.60"); // function invocation
isNaN(0); // function invocation
```

## 3.1. **this in method invocation**

> **this** is the **object that owns the method** in a method invocation

![4-1.png](http://upload-images.jianshu.io/upload_images/5518635-cba99ab07fb77da2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
var calc = {
  num: 0,
  increment: function() {
    console.log(this === calc); // => true
    this.num += 1;
    return this.num;
  }
};
// method invocation. this is calc
calc.increment(); // => 1
calc.increment(); // => 2
```

```js
class Planet {
  constructor(name) {
    this.name = name;
  }
  getName() {
    console.log(this === earth); // => true
    return this.name;
  }
}
var earth = new Planet("Earth");
// method invocation. the context is earth
earth.getName(); // => 'Earth'
```

## 3.2. **Pitfall:separating method from it object**

* A method from an object can be extracted into a separated variable **var alone = myObj.myMethod**. When the method is called alone, detached from the original object **alone()**, you might think that **this** is the object on which the method was defined.

* Correctly if the method is called without an object, then a function invocation happens: where **this** is the global object **window** or **undefined** in strict mode.

```js
function Animal(type, legs) {
  this.type = type;
  this.legs = legs;
  this.logInfo = function() {
    console.log(this === myCat); // => false
    console.log("The " + this.type + " has " + this.legs + " legs");
  };
}
var myCat = new Animal("Cat", 4);
// logs "The undefined has undefined legs"
// or throws a TypeError in strict mode
setTimeout(myCat.logInfo, 1000);
```

# 4. **Constructor invocation**

```js
function Country(name, traveled) {
  this.name = name ? name : "United Kingdom";
  this.traveled = Boolean(traveled); // transform to a boolean
}
Country.prototype.travel = function() {
  this.traveled = true;
};
// Constructor invocation
var france = new Country("France", false);
// Constructor invocation
var unitedKingdom = new Country();

france.travel(); // Travel to France
```

## 4.1. **this in constructor invocation**

> **this** is the **newly created object** in a constructor invocation

![5-1.png](http://upload-images.jianshu.io/upload_images/5518635-88043fdff1b71f15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
class Bar {
  constructor() {
    console.log(this instanceof Bar); // => true
    this.property = "Default Value";
  }
}
// Constructor invocation
var barInstance = new Bar();
barInstance.property; // => 'Default Value'
```

## 4.2. **Pitfall:forgetting about new**

```js
function Vehicle(type, wheelsCount) {
  if (!(this instanceof Vehicle)) {
    throw Error("Error: Incorrect invocation");
  }
  this.type = type;
  this.wheelsCount = wheelsCount;
  return this;
}
// Constructor invocation
var car = new Vehicle("Car", 4);
car.type; // => 'Car'
car.wheelsCount; // => 4
car instanceof Vehicle; // => true

// Function invocation. Generates an error.
var brokenCar = Vehicle("Broken Car", 3);
```

# 5. **Indirect invocation**

* The method **.call(thisArg[, arg1[, arg2[, ...]]])** accepts the first argument **thisArg** as the context of the invocation and a list of arguments **arg1, arg2, ...** that are passed as arguments to the called function.

* The method **.apply(thisArg, [arg1, arg2, ...])** accepts the first argument **thisArg** as the context of the invocation and an array-like object of values **[arg1, arg2, ...]** that are passed as arguments to the called function.

```js
function increment(number) {
  return ++number;
}
increment.call(undefined, 10); // => 11
increment.apply(undefined, [10]); // => 11
```

## 5.1. **this in indirect invocation**

> **this** is the **first argument** of **.call()** or **.apply()** in an indirect invocation

![6-1.png](http://upload-images.jianshu.io/upload_images/5518635-2a5585f668d792b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
function Runner(name) {
  console.log(this instanceof Rabbit); // => true
  this.name = name;
}
function Rabbit(name, countLegs) {
  console.log(this instanceof Rabbit); // => true
  // Indirect invocation. Call parent constructor.
  Runner.call(this, name);
  this.countLegs = countLegs;
}
var myRabbit = new Rabbit("White Rabbit", 4);
myRabbit; // { name: 'White Rabbit', countLegs: 4 }
```

# 6. **Bound function**

```js
function multiply(number) {
  "use strict";
  return this * number;
}
// create a bound function with context
var double = multiply.bind(2);
// invoke the bound function
double(3); // => 6
double(10); // => 20
```

## 6.1. **this in bound function**

> **this** is **first argument** of **.bind()** when invoking a bound function

![7-1.png](http://upload-images.jianshu.io/upload_images/5518635-0fc7097dfffaaea5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
var numbers = {
  array: [3, 5, 10],
  getNumbers: function() {
    return this.array;
  }
};
// Create a bound function
var boundGetNumbers = numbers.getNumbers.bind(numbers);
boundGetNumbers(); // => [3, 5, 10]
// Extract method from object
var simpleGetNumbers = numbers.getNumbers;
simpleGetNumbers(); // => undefined or throws an error in strict mode
```

## 6.2. **Tight context binding**

```js
function getThis() {
  "use strict";
  return this;
}
var one = getThis.bind(1);
// Bound function invocation
one(); // => 1
// Use bound function with .apply() and .call()
one.call(2); // => 1
one.apply(2); // => 1
// Bind again
one.bind(2)(); // => 1
// Call the bound function as a constructor
new one(); // => Object
```

# 7. **Arrow function**

```js
var hello = name => {
  return "Hello " + name;
};
hello("World"); // => 'Hello World'
// Keep only even numbers
[1, 2, 5, 6].filter(item => item % 2 === 0); // => [2, 6]
```

```js
var sumArguments = (...args) => {
  console.log(typeof arguments); // => 'undefined'
  return args.reduce((result, item) => result + item);
};
sumArguments.name; // => ''
sumArguments(5, 5, 6); // => 16
```

## 7.1. **this in arrow function**

> **this** is the **enclosing context** where the arrow function is defined

![8-1.png](http://upload-images.jianshu.io/upload_images/5518635-c1cfe69af918e845.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  log() {
    console.log(this === myPoint); // => true
    setTimeout(() => {
      console.log(this === myPoint); // => true
      console.log(this.x + ":" + this.y); // => '95:165'
    }, 1000);
  }
}
var myPoint = new Point(95, 165);
myPoint.log();
```

An arrow function is bound with the lexical context **once and forever**. **this** cannot be modified even if using the context modification methods:

```js
var numbers = [1, 2];
(function() {
  var get = () => {
    console.log(this === numbers); // => true
    return this;
  };
  console.log(this === numbers); // => true
  get(); // => [1, 2]
  // Use arrow function with .apply() and .call()
  get.call([0]); // => [1, 2]
  get.apply([0]); // => [1, 2]
  // Bind
  get.bind([0])(); // => [1, 2]
}.call(numbers));
```

## 7.2. **Pitfall:defining method with arrow function**

You might want to use arrow functions to declare methods on an object. Fair enough: their declaration is quite short comparing to a function expression: **(param) => {...} instead of function(param) {..}**.

```js
function Period(hours, minutes) {
  this.hours = hours;
  this.minutes = minutes;
}
Period.prototype.format = () => {
  console.log(this === window); // => true
  return this.hours + " hours and " + this.minutes + " minutes";
};
var walkPeriod = new Period(2, 30);
walkPeriod.format(); // => 'undefined hours and undefined minutes'
```
