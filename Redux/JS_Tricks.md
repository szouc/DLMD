# Cache

```js
function sum (list) {
  return list.reduce( (a, b) => a = b )
}

function memoize (fn) {
  var cache = {}
  return function (arg) {
    var hash = JSON.stringify(arg)
    if (hash in cache) {
      return cache[hash]
    }
    return cache[hash] = fn.call(this, arg)
  }
}

var memoSum = memoize(sum);

var array = [1, 2, 3, 4, 5]
memoSum(array)  // calulate 14
memoSum(array)  // get 14 from cache
array.push(6)
memoSum(array) // calulate 20
```

```js
function memoize (fn) {
  var prevArg
  var prevResult
  return function (arg) {
    if (arg === prevArg) { // Only fit for immutable value
      return prevResult
    }
    prevArg = arg
    return prevResult = fn.call(this, arg)
  }
}
```