<!-- TOC -->

- [1. 基本定义](#1-基本定义)
- [2. 被观察对象 ( `Observable` )](#2-被观察对象--observable-)
  - [2.1. 抽取与推送](#21-抽取与推送)
  - [2.2. `Observable` 是函数的泛化](#22-observable-是函数的泛化)
- [3. 深入 `Observable`](#3-深入-observable)
  - [3.1. 创建 `Observable`](#31-创建-observable)
  - [3.2. 订阅 `Observable`](#32-订阅-observable)

<!-- /TOC -->

# 1. 基本定义

ReactiveX 是一种融合了观察者模式、迭代器模式和函数式编程思想的异步编程接口，她的 JS 实现是 RxJS 库。RxJS 就是利用被观测量 ( observable sequences ) 来构建**异步**和**事件驱动**程序。

> Think of RxJS as Lodash for events

RxJS 中关键的几个概念都是用来解决异步事件管理任务的：

* **被观察对象 ( `Observable` )**：未来被调用的值或事件的抽象。
* **观察者 ( `Observer` )**：回调函数的集合，用于监听和处理被观察者推送过来的值。
* **订阅 ( `Subscription` )**：代表被观测者的执行过程 ，主要用于取消执行过程。
* **主题 ( `Subject` )**：类似于事件发布，将一个值或事件多播给多个观测者的唯一方法。

# 2. 被观察对象 ( `Observable` )

以往 JS 语言中对数据处理的手段如下表：

|          | **单值**   | **多值**   |
| -------- | ---------- | ---------- |
| **抽取** | `Function` | `Iterator` |
| **推送** | `Promise`  |            |

`Observables` 则可以对多值进行懒推送。

**例子**:

```js
/**
 * Observable 首先推送 1，2，3 他们被订阅时会立即执行（同步），
 * 然后推送一个被订阅后延迟 1 秒执行的异步数据 4 ：
 */
var observable = Rx.Observable.create(function(observer) {
  observer.next(1)
  observer.next(2)
  observer.next(3)
  setTimeout(() => {
    observer.next(4)
    observer.complete()
  }, 1000)
})

// Observable 对象提供 `subscribe` 方法用以进行订阅。
console.log("just before subscribe")
observable.subscribe({
  // observer 对象
  next: x => console.log("got value " + x),
  error: err => console.error("something wrong occurred: " + err),
  complete: () => console.log("done")
}) // 类似 Promise ， 但可以推送多值
console.log("just after subscribe")

// 执行结果
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

## 2.1. 抽取与推送

抽取和推送是数据在生产者和消费者之间传输的两种协议，具体如下表：

|          | **生产者**                        | **消费者**                    |
| -------- | --------------------------------- | ----------------------------- |
| **抽取** | **被动：** 被调用时生产数据       | **主动：** 决定何时调用       |
| **推送** | **主动：** 按照自己的方式生产数据 | **被动：** 仅仅对数据进行操作 |

* **函数 ( Function )** 懒加载运行并在被调用时同步返回单值。
* **生成器对象 ( generator )** 懒加载运行并在迭代过程中同步返回零到无穷个值。
* **Promise** 运行后最终返回不可变的单状态 ( `resolve` 或 `reject` )。
* **被观察者 ( Observable )** 懒加载运行并在被调用开始异步或同步返回多值。

## 2.2. `Observable` 是函数的泛化

> Observables are like functions with zero arguments, but generalize those to allow multiple values. ( `Observables` 像一些无参函数，但比函数更一般化，可以返回多个值。)

```js
// Function 懒加载，不会执行。
function foo() {
  console.log("Hello")
  return 42
}

var x = foo.call() // same as foo()
console.log(x)
var y = foo.call() // same as foo()
console.log(y)

// Observable 懒加载，不会执行。
var foo = Rx.Observable.create(function(observer) {
  console.log("Hello")
  observer.next(42)
})

foo.subscribe(function(x) {
  console.log(x)
})
foo.subscribe(function(y) {
  console.log(y)
})
```

> 1. 每一个 `Observable` 的 `subscribe` 方法都会触发各自独立的副作用。这与 `EventEmitter` 共享副作用不同。
>
> 2. Subscribing to an Observable is analogous to calling a Function. (订阅一个 `Observable` 类似于函数的调用。)
>
> 3. Observables are able to deliver values either synchronously or asynchronously. ( `Observables` 可以通过异步或同步方式推送数据。)

**`Observables` 可以返回多个值：**

```js
var foo = Rx.Observable.create(function(observer) {
  console.log("Hello")
  observer.next(42)
  observer.next(100)
  observer.next(200)
  setTimeout(() => {
    observer.next(300) // happens asynchronously
  }, 1000)
})

console.log("before")
foo.subscribe(function(x) {
  console.log(x)
})
console.log("after"):
```

**总结：**

* `func.call()` 只能同步返回一个值。
* `observable.subscribe()` 可以通过异步或同步方式返回多个值。

# 3. 深入 `Observable`

`Observable` 的核心问题：

* **创建 ( `Creating` )** `Observable`
* **订阅 ( `Subscribing` )** `Observable`
* **执行 ( `Executing` )** `Observable`
* **销毁 ( `Disposing` )** `Observable`

## 3.1. 创建 `Observable`

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  var id = setInterval(() => {
    observer.next("hi")
  }, 1000)
})
```

> Observable can be create with `create`, but usually we use the so-called [creation operators](http://reactivex.io/rxjs/manual/overview.html#creation-operators),like `of`, `from`, `interval`, etc.

## 3.2. 订阅 `Observable`
