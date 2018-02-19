<!-- TOC -->

- [1. 基本定义](#1-%E5%9F%BA%E6%9C%AC%E5%AE%9A%E4%B9%89)
- [2. 被观察对象 ( Observable )](#2-%E8%A2%AB%E8%A7%82%E5%AF%9F%E5%AF%B9%E8%B1%A1-observable)
  - [2.1. 抽取与推送](#21-%E6%8A%BD%E5%8F%96%E4%B8%8E%E6%8E%A8%E9%80%81)
  - [2.2. Observable 是函数的泛化](#22-observable-%E6%98%AF%E5%87%BD%E6%95%B0%E7%9A%84%E6%B3%9B%E5%8C%96)
- [3. 深入 Observable](#3-%E6%B7%B1%E5%85%A5-observable)
  - [3.1. 创建 Observable](#31-%E5%88%9B%E5%BB%BA-observable)
  - [3.2. 订阅 Observable](#32-%E8%AE%A2%E9%98%85-observable)
  - [3.3. 执行 Observable](#33-%E6%89%A7%E8%A1%8C-observable)
  - [3.4. 终止 Observable 执行](#34-%E7%BB%88%E6%AD%A2-observable-%E6%89%A7%E8%A1%8C)
- [4. 观察者 (Observer)](#4-%E8%A7%82%E5%AF%9F%E8%80%85-observer)
- [5. 订阅 (Subscription)](#5-%E8%AE%A2%E9%98%85-subscription)
- [6. 主题 (Subject)](#6-%E4%B8%BB%E9%A2%98-subject)
  - [6.1. 多播 Observables](#61-%E5%A4%9A%E6%92%AD-observables)
  - [6.2. 引用计数](#62-%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0)
  - [6.3. BehaviorSubject](#63-behaviorsubject)
  - [6.4. ReplaySubject](#64-replaysubject)
  - [6.5. AsyncSubject](#65-asyncsubject)
- [7. 操作符 (Operators)](#7-%E6%93%8D%E4%BD%9C%E7%AC%A6-operators)
  - [7.1. 何为操作符？](#71-%E4%BD%95%E4%B8%BA%E6%93%8D%E4%BD%9C%E7%AC%A6%EF%BC%9F)
  - [7.2. 实例操作符与静态操作符](#72-%E5%AE%9E%E4%BE%8B%E6%93%8D%E4%BD%9C%E7%AC%A6%E4%B8%8E%E9%9D%99%E6%80%81%E6%93%8D%E4%BD%9C%E7%AC%A6)
  - [7.3. Marble 图](#73-marble-%E5%9B%BE)
- [8. 调度器 (Scheduler)](#8-%E8%B0%83%E5%BA%A6%E5%99%A8-scheduler)

<!-- /TOC -->

# 1. 基本定义

ReactiveX 是一种融合了观察者模式、迭代器模式和函数式编程思想的异步编程接口，她的 JS 实现是 RxJS 库。RxJS 就是利用被观测量 ( observable sequences ) 来构建**异步**和**事件驱动**程序。

> Think of RxJS as Lodash for events

RxJS 中关键的几个概念都是用来解决异步事件管理任务的：

* **被观察对象 ( Observable )**：未来被调用的值或事件的抽象。
* **观察者 ( Observer )**：回调函数的集合，用于监听和处理被观察者推送过来的值。
* **订阅 ( Subscription )**：代表被观察者的执行过程 ，主要用于取消执行过程。
* **主题 ( Subject )**：类似于事件发布，将一个值或事件多播给多个观察者的唯一方法。

# 2. 被观察对象 ( Observable )

以往 JS 语言中对数据处理的手段如下表：

|          | **单值**   | **多值**   |
| -------- | ---------- | ---------- |
| **抽取** | `Function` | `Iterator` |
| **推送** | `Promise`  |            |

Observables 则可以对多值进行懒推送。

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

## 2.2. Observable 是函数的泛化

> Observables are like functions with zero arguments, but generalize those to allow multiple values. ( Observables 像一些无参函数，但比函数更一般化，可以返回多个值。)

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

> 1. 每一个 Observable 的 `subscribe` 方法都会触发各自独立的副作用。这与 EventEmitter 共享副作用不同。
>
> 2. Subscribing to an Observable is analogous to calling a Function. (订阅一个 Observable 类似于函数的调用。)
>
> 3. Observables are able to deliver values either synchronously or asynchronously. ( Observables 可以通过异步或同步方式推送数据。)

**Observables 可以返回多个值：**

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

# 3. 深入 Observable

Observable 的核心问题：

* **创建 ( Creating )** Observable
* **订阅 ( Subscribing )** Observable
* **执行 ( Executing )** Observable
* **回收 ( Disposing )** Observable

## 3.1. 创建 Observable

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  var id = setInterval(() => {
    observer.next("hi")
  }, 1000)
})
```

> Observable can be create with `create`, but usually we use the so-called [creation operators](http://reactivex.io/rxjs/manual/overview.html#creation-operators),like `of`, `from`, `interval`, etc.

## 3.2. 订阅 Observable

上例中的 `observable` 可通过 `subscribe` 方法进行订阅：

```js
observable.subscribe(x => console.log(x))
```

这里特意将 `Rx.Observable.create(function subscribe(observe){...})` 函数名书写成 `subscribe` 方法名，主要是为了说明当一个特定的 Observer 调用了 `subscribe` 方法，就将这个 Observer 传递进 `Rx.Observable.create(function subscribe(observe){...})` 函数中。同时也可以看出订阅同一个 Observable 的 Observer 都有各自的执行程序。

> Subscribing to an Observable is like calling a function, providing callbacks where the data will be delivered to. (订阅 Observable 类似于调用函数，即对 Observable 推送的数据提供回调函数。)

此外不同于 `addEventListener` / `removeEventListener` , Observer 并没有注册任何信息到 Observable 中。

**总结：** `subscribe` 方法就是先执行 Observable 的程序体，然后将推送出来的数据或事件发送给 Observer 中执行。

## 3.3. 执行 Observable

`Rx.Observable.create(function subscribe(observe){...})` 可以看作为 Observable 的程序体，只有在 Observer 订阅时进行执行。程序体可以推送多个数据，既可以是同步推送也可以异步推送。在 Observable 程序体中有三种推送方式：

* **Next 推送：** 可以推送 Number，String，Object 等。
* **Error 推送：** 推送一个异常。
* **Complete 推送：** 仅仅是一个完成通知，没有数据的推送。

> In an Observable Execution, zero to infinite Next notifications may be delivered. If either an Error or Complete notification is delivered, then nothing else can be delivered afterwards. (在 Observable 程序体中可以有多个 Next 推送， 但只能有一个 Error 推送**或者** Complete 推送，并且之后不会再有推送起效。)

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  try {
    observer.next(1)
    observer.next(2)
    observer.next(3)
    observer.complete()
  } catch (err) {
    observer.error(err) // delivers an error if it caught one
  }
})
```

## 3.4. 终止 Observable 执行

因为 Observable 程序体有可能是一个无限序列，所以必须提供一个接口来终止程序体的执行。首先这里的终止是针对特定 Observer 定义的，因为 Observable 都依赖于特定的 Observer 执行，即没有 Observer 的订阅，也就不会执行 Observable 程序体。可以认为终止 Observable 执行就是特定的 Observer 停止了对 Observable 的订阅。

当 `observable.subscribe` 方法被调用时，会创建并执行一个新的 Observable 程序体副本，并且该方法会返回一个对象 Subscription :

```js
var subscription = observable.subscribe(x => console.log(x))
```

> When you subscribe, you get back a Subscription, which represents the ongoing execution. Just call unsubscribe() to cancel the execution. (调用 `subscribe` 方法会返回一个 Subscription 对象，该对象代表正在执行的 Observable 程序体，只需要调用该对象的 `unsubscribe` 方法即可终止程序体执行。)

```js
// 也可以自定义 unsubscribe 方法
var observable = Rx.Observable.create(observer => {
  var intervalID = setInterval(() => {
    observer.next("hi")
  }, 1000)
  // Provide a way of canceling and disposing the interval resource
  return function unsubscribe() {
    clearInterval(intervalID)
  }
})

var subscriptionA = observable.subscribe(x => console.log(`A_${x}`))
var subscriptionB = observable.subscribe(x => console.log(`B_${x}`))

subscriptionA.unsubscribe() // 循环输出：B_hi，因为 A 取消了订阅。
```

# 4. 观察者 (Observer)

Observer 是 Observable 推送数据的消费方，可以简单的认为 Observer 是一个回调函数集，用来响应 Observable 传递来的三种推送。

```js
var observer = {
  next: x => console.log("Observer got a next value: " + x),
  error: err => console.error("Observer got an error: " + err),
  complete: () => console.log("Observer got a complete notification")
}
```

之前已提到 Observer 使用 `subscribe` 方法订阅 Observable ：

```js
observable.subscribe(observer)
```

> Observers are just objects with three callbacks, one for each type of notification that an Observable may deliver. ( Observer 对象提供了三种回调函数，响应来自 Observable 的三种推送。)

# 5. 订阅 (Subscription)

Subscription 对象表示一个可回收资源，通常指 Observable 程序体。

```js
var observable = Rx.Observable.interval(1000)
var subscription = observable.subscribe(x => console.log(x))
// Later:
// This cancels the ongoing Observable execution which
// was started by calling subscribe with an Observer.
subscription.unsubscribe()
```

> A Subscription essentially just has an unsubscribe() function to release resources or cancel Observable executions. ( Subscription 对象拥有一个 `unsubscribe` 方法，该方法可以释放资源或终止 Observable 程序体执行。)

Subscription 对象可以进行组合，这样只需调用一个 `unsubscribe` 方法就可以回收多个资源:

```js
var observable1 = Rx.Observable.interval(400)
var observable2 = Rx.Observable.interval(300)

var subscription = observable1.subscribe(x => console.log("first: " + x))
var childSubscription = observable2.subscribe(x => console.log("second: " + x))

subscription.add(childSubscription)

setTimeout(() => {
  // Unsubscribes BOTH subscription and childSubscription
  subscription.unsubscribe()
}, 1000)
```

Subscription 对象还有 `remove` 方法，用来移除子 Subscription 对象。

# 6. 主题 (Subject)

Subject 是一种特殊的 Observable ，她可以将数据多播给多个 Observers 。 普通的 Observable 是单播的（每一个 Observer 接收所订阅的 Observable 推送，正如之前所说每次订阅都创建一个 Observable 程序体副本来推送数据），而 Subject 是多播的。

> A Subject is like an Observable, but can multicast to many Observers. Subjects are like EventEmitters: they maintain a registry of many listeners. (Subject 类似于 Observable，但可以将数据多播给多个 Observer，Subject 也类似于 EventEmitter，她维持一个注册信息表。)

**Subject 是 Observable 。** 所有的 Subject 都可以被 Observer `subscribe` (订阅) ，根据之前所描述的， Observer 无法获知推送来的数据是来自 Observable 还是 Subject 。

深入 Subject 可以发现 `subscribe` 方法并不会创建一个程序体副本（与 Observable 的区别），她将订阅的 Observers 注册进一个信息表，类似于 `addListener` 。

**Subject 是 Observer 。** Subject 对象也拥有 Observer 对象中的 `next(v)`，`error(e)`，`complete()` 三种方法。Observable 中调用 `next(newValue)` 就可推送数据到 Subject ，然后 Subject 会将数据多播给注册的 Observers 。

```js
var subject = new Rx.Subject()

subject.subscribe({
  next: v => console.log("observerA: " + v)
})
subject.subscribe({
  next: v => console.log("observerB: " + v)
})

var observable = Rx.Observable.from([1, 2, 3])

observable.subscribe(subject) // You can subscribe providing a Subject
```

Subject 可作为 Observable 和 Observer 之间的中继器，订阅 Observable 获取推送数据，将数据多播给注册的 Observers。也就是 Subject 可以将单播的 Observable 转化为多播，这也是唯一一种将 Observable 推送的数据多播给多个 Observers 的方法。

有三种特定的 Subject：`BehaviorSubject`，`ReplaySubject`，`AsyncSubject` 。

## 6.1. 多播 Observables

将 Subject 作为中继器可生成一种新的 Observable，称为 Multicasted Observable 。

```js
var source = Rx.Observable.from([1, 2, 3])
var subject = new Rx.Subject()
var multicasted = source.multicast(subject)

// These are, under the hood, `subject.subscribe({...})`:
multicasted.subscribe({
  next: v => console.log("observerA: " + v)
})
multicasted.subscribe({
  next: v => console.log("observerB: " + v)
})

// This is, under the hood, `source.subscribe(subject)`:
multicasted.connect()
```

`multicast` 方法返回 ConnectableObservable 对象，该对象拥有 `connect` 方法。`connect()` 方法起到 `source.subscribe(subject)` 作用，因此返回一个 Subscription 对象，可以终止 Observable 程序体执行。

## 6.2. 引用计数

操作 `connect` 方法进行连接和终止显得过于繁琐。是否有一种机制使得第一个 Observer 订阅时自动连接 Observable，最后一个 Observer 终止订阅时自动断开 Observable ？

> refCount makes the multicasted Observable automatically start executing when the first subscriber arrives, and stop executing when the last subscriber leaves. (refCount 实现了第一个 Observer 订阅时自动连接，最后一个 Observer 取消订阅时自动断开机制。)

```js
var source = Rx.Observable.interval(500)
var subject = new Rx.Subject()
var refCounted = source.multicast(subject).refCount()
var subscription1, subscription2, subscriptionConnect

// This calls `connect()`, because
// it is the first subscriber to `refCounted`
console.log("observerA subscribed")
subscription1 = refCounted.subscribe({
  next: v => console.log("observerA: " + v)
})

setTimeout(() => {
  console.log("observerB subscribed")
  subscription2 = refCounted.subscribe({
    next: v => console.log("observerB: " + v)
  })
}, 600)

setTimeout(() => {
  console.log("observerA unsubscribed")
  subscription1.unsubscribe()
}, 1200)

// This is when the shared Observable execution will stop, because
// `refCounted` would have no more subscribers after this
setTimeout(() => {
  console.log("observerB unsubscribed")
  subscription2.unsubscribe()
}, 2000)
```

`refCount` 方法仅存在 ConnectableObservable 中，并且返回一个 Observable 。

## 6.3. BehaviorSubject

BehaviorSubject 拥有**当前值**的定义，她保留推送给消费者的最后一个值，当有新的 Observer 订阅时，立即将该值推送给新的 Observer 。

> BehaviorSubjects are useful for representing "values over time". For instance, an event stream of birthdays is a Subject, but the stream of a person's age would be a BehaviorSubject. (BehaviorSubjects 有效的保留了“持续值”。比如生日的事件流是一个 Subject，而年龄则是一个 BehaviorSubject 。)

```js
var subject = new Rx.BehaviorSubject(0) // 0 is the initial value

subject.subscribe({
  next: v => console.log("observerA: " + v)
})

subject.next(1)
subject.next(2)

subject.subscribe({
  next: v => console.log("observerB: " + v)
})

subject.next(3)
```

## 6.4. ReplaySubject

ReplaySubject 可以记录一些之前的数据，并将数据发送给新的订阅者。

> A ReplaySubject records multiple values from the Observable execution and replays them to new subscribers. (ReplaySubject 从 Observable 程序体中记录了多个数据，并将这些数据回放给新的订阅者。)

```js
var subject = new Rx.ReplaySubject(3) // buffer 3 values for new subscribers

subject.subscribe({
  next: v => console.log("observerA: " + v)
})

subject.next(1)
subject.next(2)
subject.next(3)
subject.next(4)

subject.subscribe({
  next: v => console.log("observerB: " + v)
})

subject.next(5)
```

除了缓存大小外，还可以定义窗口时限：

```js
var subject = new Rx.ReplaySubject(100, 500 /* windowTime */)

subject.subscribe({
  next: v => console.log("observerA: " + v)
})

var i = 1
setInterval(() => subject.next(i++), 200)

setTimeout(() => {
  subject.subscribe({
    next: v => console.log("observerB: " + v)
  })
}, 1000)
```

## 6.5. AsyncSubject

AsyncSubject 会在 Observable 程序体执行完毕后将最终值推送给订阅的 Observers 。

```js
var subject = new Rx.AsyncSubject()

subject.subscribe({
  next: v => console.log("observerA: " + v)
})

subject.next(1)
subject.next(2)
subject.next(3)
subject.next(4)

subject.subscribe({
  next: v => console.log("observerB: " + v)
})

subject.next(5)
subject.complete()
```

# 7. 操作符 (Operators)

尽管 Observable 是 RxJS 的基础，但是 RxJS 中最有用的还是操作符，她以声明的方式将复杂的异步操作轻易的组合起来。

## 7.1. 何为操作符？

操作符是 Observable 对象的**方法**，如 `.map(...)`，`.filter(...)`，`.merge(...)` 等。调用操作符时，她们不会改变当前的 Observable 实例，而会返回一个新的 Observable 实例，新实例的订阅机制继承自之前的实例。

> An Operator is a function which creates a new Observable based on the current Observable. This is a pure operation: the previous Observable stays unmodified. (操作符会根据当前 Observable，创建一个新的 Observable。这是纯函数的理念：之前的 Observable 没有改变。)

操作符是一种纯函数，在不改变输入的 Observable 情况下，利用输入 Observable 产生新的输出 Observable，订阅了输出的 Observable 同时也会订阅输入的 Observable 。

```js
function multiplyByTen(input) {
  var output = Rx.Observable.create(function subscribe(observer) {
    input.subscribe({
      next: v => observer.next(10 * v),
      error: err => observer.error(err),
      complete: () => observer.complete()
    })
  })
  return output
}

var input = Rx.Observable.from([1, 2, 3, 4])
var output = multiplyByTen(input)
output.subscribe(x => console.log(x))
```

订阅输出 Observable 的同时会导致订阅输入 Observable，这种方式成为 “操作符订阅链 (operator subscription chain)”

## 7.2. 实例操作符与静态操作符

一般谈到操作符，都默认是实例操作符，也就是操作符方法在 Observable 实例上。

```js
Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
  var input = this
  return Rx.Observable.create(function subscribe(observer) {
    input.subscribe({
      next: v => observer.next(10 * v),
      error: err => observer.error(err),
      complete: () => observer.complete()
    })
  })
}
```

> Instance operators are functions that use the this keyword to infer what is the input Observable. (实例操作符使用 this 关键字来指代当前的输入 Observable 。)

不同于实例操作符，静态操作符直接存在于 Observable 类中。静态操作符内部没有 this 关键字，取而代之的是完整的参数传递。

> Static operators are pure functions attached to the Observable class, and usually are used to create Observables from scratch. (静态操作符是类上的纯函数，通常用于构建 Observables 。)

最常见的静态操作符就是构建操作符 (Creation Operators)，她没有输入输出的转换，只是简单的利用不可观察值构建新的 Observable 。

```js
var observable = Rx.Observable.interval(1000 /* number of milliseconds */)
```

然而并不是所有的静态操作符都是简单的进行构建，一些组合操作符 (Combination Operators) 也可能是静态的，比如 `merge`，`combineLatest`，`concat` 等。

```js
var observable1 = Rx.Observable.interval(1000)
var observable2 = Rx.Observable.interval(400)

var merged = Rx.Observable.merge(observable1, observable2)
```

## 7.3. Marble 图

[学习 Marble 图](https://medium.com/@jshvarts/read-marble-diagrams-like-a-pro-3d72934d3ef5)

文字解释操作符的流程显然是不够的，许多操作符都与时间相关，比如延迟，采样，节流，防抖。图表是一个更好的选择，Marble 图将操作符的工作流程可视化，并且还包含了输入 Observables，操作符和她的参数以及输出 Observable 的表示。

> In a marble diagram, time flows to the right, and the diagram describes how values ("marbles") are emitted on the Observable execution. (在 Marble 图中，时间朝右方向增加，并且描述了 Observable 程序体推送的数值。)

下表是 Marble 图的符号意义：

| 符号 | 意义                                                         |
| ---- | ------------------------------------------------------------ |
| ●    | 可观察序列推送的单值。(from IObserver<T>.OnNext(T))          |
| ▲    | 某种方式转化后的数值。                                       |
| x    | 可观察序列的异常推送。(from IObserver<T>.OnError(Exception)) |
| \|   | 可观察序列完成推送。(from IObserver<T>.OnCompleted())        |

具体操作符请参看[官网](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)。

# 8. 调度器 (Scheduler)

调度器作用于订阅开始阶段和推送阶段，她包含三个部分：

* **调度器是数据结构。** 调度器根据优先原则或其他准则存储和排列任务。
* **调度器是执行环境。** 调度器定义了任务何时何地执行。
* **调度器有时钟。** 调度器通过 `now` 方法提供了一个“时间”概念。在特定的调度器上任务必须遵守该时钟的“时间”概念。

> A Scheduler lets you define in what execution context will an Observable deliver notifications to its Observer. (调度器定义了 Observable 推送数据到 Observer 的执行环境。)

```js
var observable = Rx.Observable.create(function(observer) {
  observer.next(1)
  observer.next(2)
  observer.next(3)
  observer.complete()
}).observeOn(Rx.Scheduler.async)

console.log("just before subscribe")
observable.subscribe({
  next: x => console.log("got value " + x),
  error: err => console.error("something wrong occurred: " + err),
  complete: () => console.log("done")
})
console.log("just after subscribe")
```

输出的结果：

```
just before subscribe
just after subscribe
got value 1
got value 2
got value 3
done
```

这是因为 `observeOn(Rx.Scheduler.async)` 在 `Observable.create` 和最终的 Observer 中添加了一个代理 Observer 。
