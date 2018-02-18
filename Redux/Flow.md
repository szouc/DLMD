## Literal Types Mixed Types

In general, programs have several different categories of types:

**A single type:**

Here the input value can only be a number.

```js
function square(n: number) {
  return n * n;
}
```

**A group of different possible types:**

Here the input value could be either a string or a number.

```js
function stringifyBasicValue(value: string | number) {
  return '' + value;
}
```

**A type based on another type:**

Here the return type will be the same as the type of whatever value is passed into the function.

```js
function identity<T>(value: T): T {
  return value;
}
```

These three are the most common categories of types. They will make up the majority of the types you’ll be writing.

However, there is also a fourth category.

An arbitrary type that could be anything:

Here the passed in value is an unknown type, it could be any type and the function would still work.

```js
function getTypeOf(value: mixed): string {
  return typeof value;
}
```

These unknown types are less common, but are still useful at times.

You should represent these values with mixed.

Anything goes in, Nothing comes out 

mixed will accept any type of value. Strings, numbers, objects, functions– anything will work.

```js
// @flow
function stringify(value: mixed) {
  // ...
}

stringify("foo");
stringify(3.14);
stringify(null);
stringify({});
```

**When you try to use a value of a mixed type you must first figure out what the actual type is or you’ll end up with an error.**

```js
// @flow
function stringify(value: mixed) {
  // $ExpectError
  return "" + value; // Error!
}

stringify("foo");
```

Instead you must ensure the value is a certain type by refining it.

```js
// @flow
function stringify(value: mixed) {
  if (typeof value === 'string') {
    return "" + value; // Works!
  } else {
    return "";
  }
}

stringify("foo");
```

Because of the typeof value === 'string' check, Flow knows the value can only be a string inside of the if statement. This is known as a refinement.

## Literal Types Any Types

There are only a couple of scenarios where you might consider using any:

- When you are in the process of converting a existing code to using Flow types and you are currently blocked on having the code type checked (maybe other code needs to be converted first).
- When you are certain your code works and for some reason Flow is unable to type check it correctly. There are a (decreasing) number of idioms in JavaScript that Flow is unable to statically type.

## Function Types

### Syntax of functions

There are three forms of functions that each have their own slightly different syntax.

- Function Declarations

Here you can see the syntax for function declarations with and without types added.

```js
function method(str, bool, ...nums) {
  // ...
}

function method(str: string, bool?: boolean, ...nums: Array<number>): void {
  // ...
}
```

- Arrow Functions

Here you can see the syntax for arrow functions with and without types added.

```js
let method = (str, bool, ...nums) => {
  // ...
};

let method = (str: string, bool?: boolean, ...nums: Array<number>): void => {
  // ...
};
```

- Function Types

Here you can see the syntax for writing types that are functions.

```js
(str: string, bool?: boolean, ...nums: Array<number>) => void
```

You may also optionally leave out the parameter names.

```js
(string, boolean | void, Array<number>) => void
```

You might use these functions types for something like a callback.

```js
function method(callback: (error: Error | null, value: string | null) => void) {
  // ...
}
```

Sometimes it is useful to write types that accept arbitrary functions, for those you should write ```() => mixed``` like this:

```js
function method(func: () => mixed) {
  // ...
}
```

However, if you need to opt-out of the type checker, and don’t want to go all the way to **any**, you can instead use **Function**. **Function is unsafe and should be avoided.**

## Object Types

### Sealed objects 

When you create an object with its properties, you create a sealed object type in Flow. These sealed objects will know all of the properties you declared them with and the types of their values.

```js
// @flow
var obj = {
  foo: 1,
  bar: true,
  baz: 'three'
};

var foo: number  = obj.foo; // Works!
var bar: boolean = obj.bar; // Works!
// $ExpectError
var baz: null    = obj.baz; // Error!
var bat: string  = obj.bat; // Error!
```

But when objects are sealed, Flow will not allow you to add new properties to them.

```js
// @flow
var obj = {
  foo: 1
};

// $ExpectError
obj.bar = true;    // Error!
// $ExpectError
obj.baz = 'three'; // Error!
```

The workaround here might be to turn your object into an unsealed object.

### Unsealed objects 

When you create an object without any properties, you create an unsealed object type in Flow. These unsealed objects will not know all of their properties and will allow you to add new ones.

```js
// @flow
var obj = {};

obj.foo = 1;       // Works!
obj.bar = true;    // Works!
obj.baz = 'three'; // Works!
```

The inferred type of the property becomes what you set it to.

```js
// @flow
var obj = {};
obj.foo = 42;
var num: number = obj.foo;
```

#### Unknown property lookup on unsealed objects is unsafe 

Unsealed objects allow new properties to be written at any time. Flow ensures that reads are compatible with writes, but does not ensure that writes happen before reads (in the order of execution).

This means that reads from unsealed objects with no matching writes are never checked. This is an unsafe behavior of Flow which may be improved in the future.

```js
var obj = {};

obj.foo = 1;
obj.bar = true;

var foo: number  = obj.foo; // Works!
var bar: boolean = obj.bar; // Works!
var baz: string  = obj.baz; // Works?
```

### Exact object types

In Flow, it is considered safe to pass an object with extra properties where a normal object type is expected.

```js
// @flow
function method(obj: { foo: string }) {
  // ...
}

method({
  foo: "test", // Works!
  bar: 42      // Works!
});
```

Note: This is because of “width subtyping”.

Sometimes it is useful to disable this behavior and only allow a specific set of properties. For this, Flow supports “exact” object types.

```js
{| foo: string, bar: number |}
```

Unlike regular object types, it is not valid to pass an object with “extra” properties to an exact object type.

```js
// @flow
var foo: {| foo: string |} = { foo: "Hello", bar: "World!" }; // Error!
```

