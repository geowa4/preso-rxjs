footer: George Adams IV
slidenumbers: true

# Reactive
# Extensions
# for
# JavaScript

^Tonight, I'll be talking about the Reactive Extensions for JavaScript, a.k.a. RxJS.

---

# RxJS

^I don't consider myself an expert since I haven't been using it for too long—a few months.

---

![](https://upload.wikimedia.org/wikipedia/commons/c/c7/Marco_Saporiti_Atmosphere_of_Earth.png)

^My goal is to go from an exospheric view to about a stratospheric level.

---

## Multilingual

 - .NET
 - Java
 - Android
 - C/C++
 - Python
 - JavaScript

^One of the big selling points for me is that there are language-specific ports for .NET, Java, JavaScript, and many others.

^That means you learn once, write anywhere.

---

## Let's play a game.

^I'd like to start with a fun little exercise.
<Pick two people.>
<Looking at the first person.>
  I'm going to say a bunch of numbers.
  As soon as I say one, I need you to double it and say it aloud.
<Looking at the second person.>
  As soon as he says his number, you should subtract one and say it aloud.
1; 2; 3; 5; 8.
Good work.
Now, I need one more person.
<Looking at the third person.>
  When <the first person> says his number, add one to it.
  While you're not racing <the second person>, be eager.
<Looking at first and second people.>
  You're directions are unchanged.
2; 4; 6; 8; 16.
Ok, now that we have the hang of thigs, let's get still more complicated: two data sources.
<Looking at the first person.>
  I need you to say the first few even numbers.
  Just so we don't create a cacophony that no one can follow, say your number on my signal.
<Looking at the second person.>
  Subtract one from his numbers like before.
<Looking at the third person.>
  Add one to my numbers like you did before.
I need one more.
<Looking at fourth person.>
  Whenever either <the second person> or <the third person> say a number, say both of the latest numbers from each of them.
  You should always be saying two numbers.
  Both must have a value for you to say anything.
1; signal (2); 2; 3; signal (4); signal (8); 5; 8.
<combined should be (2, 1), (3, 1), (4, 1), (4, 3), (4, 7), (6, 7), (9, 7).>

---

## Data-First Development

^Great work!
That's the Reactive Extensions.
You start by thinking about your data sources.
Then, you think about the transformations that act on those sources.
Finally, you fork them and combine them to suit your needs.

---

## Types

 - Observable
 - Observer
 - Subject

^There are three main types in Rx: Observable, Observer, and Subject.

---

## Observable

^Observable is what we've seen so far as those streams of events.

```javascript
stream
  .filter(x => x > 0)
  .map(x => x * 2)
  .bufferWithTime(500)
  .subscribe(onNext, onError, onComplete)
```

---

## Observer

```javascript
stream
  .filter(x => x > 0)
  .map(x => x * 2)
  .bufferWithTime(500)
  .subscribe(myObserver)
```

^An Observer is a type that can be passed to the `subscribe` function and handles the same three cases.

---

## Observer

```javascript
myObserver.onNext(42)
myObserver.onComplete()
```

^You can also call it's methods directly.
I haven't really made Observers, but one use case could be Google Analytics.

---

## Subject

```javascript
subject = new BehaviorSubject(42)
subject.subscribe(x => console.log(x))
// 42
subject.onNext(56)
// 56
console.log(subject.getValue())
// 56
```

^However, I do use the Subject type, which implements both Observer and Observable.
Let's take a look at the BehaviorSubject, specifically.
The BehaviorSubject must have a current value.
So when you create it, it must be given an initial value (or state).

---

## Bridging to Imperative Code

```javascript
list = subject.getValue()
newList = list.push(newItem)
subject.onNext(newList)
```

^I use this type as a bridge between FRP and imperative.
Imagine a list of proposals.
You create a new one.
The server can either return an entire list of them back to you, or
you can keep the list in a BehaviorSubject and append.

---

## Bridging to Events

```javascript
var result = document.getElementById('result')

var source = Rx.Observable.fromEvent(document, 'mousemove')

var subscription = source.subscribe(function (e) {
  result.innerHTML = e.clientX + ', ' + e.clientY
})
```

^Being JavaScript, there are certain things we can do specific to the runtime.
Here we see an easy way to create a stream of DOM Events.

---

## Bridging to Callbacks

```javascript
var Rx = require('rx'),
    fs = require('fs')

// Wrap the exists method
var exists = Rx.Observable.fromCallback(fs.exists)

var source = exists('file.txt')

// Get the first argument only which is true/false
var subscription = source.subscribe(
  function (x) { console.log('onNext: %s', x) },
  function (e) { console.log('onError: %s', e) },
  function ()  { console.log('onCompleted') }
)

// => onNext: true
// => onCompleted
```

^No one likes working with callbacks so let's make those a stream of one.

---

## Bridging to Node Callbacks

```javascript
var fs = require('fs'),
    Rx = require('rx')

// Wrap fs.rename
var rename = Rx.Observable.fromNodeCallback(fs.rename)

// Rename file which returns no parameters except an error
var source = rename('file1.txt', 'file2.txt')

var subscription = source.subscribe(
  function (x) { console.log('onNext: success!') },
  function (e) { console.log('onError: %s', e) },
  function ()  { console.log('onCompleted') }
)

// => onNext: success!
// => onCompleted
```

^Node has created a de facto standard with callbacks so we can easily convert that to a stream of one.

---

## Bridging to Promises

```javascript
// Create a promise which resolves 42
var promise1 = new Promise(function (resolve, reject) {
  resolve(42)
})

var source1 = Rx.Observable.fromPromise(promise1)

var subscription1 = source1.subscribe(
  function (x) { console.log('onNext: %s', x) },
  function (e) { console.log('onError: %s', e) },
  function () { console.log('onCompleted') }
)

// => onNext: 42
// => onCompleted
```

^The astute observer may be thinking that Promises are streams of one.

---

## Bridging to Promises

```javascript
// Create a promise which rejects with an error
var promise2 = new Promise(function (resolve, reject) {
  reject(new Error('reason'))
})

var source2 = Rx.Observable.fromPromise(promise2)

var subscription2 = source2.subscribe(
  function (x) { console.log('onNext: %s', x) },
  function (e) { console.log('onError: %s', e) },
  function () { console.log('onCompleted') }
)

// => onError: reject
```

^Here we see what happens upon rejection.

---

## Bridging to Promises

```javascript
var source1 = Rx.Observable.just(1).toPromise()

source1.then(
  function (value) {
    console.log('Resolved value: %s', value)
  },
  function (reason) {
    console.log('Rejected reason: %s', reason)
  }
)

// => Resolved value: 1
```

^To Rx, Promises are just Observables, and you can convert in either direction.
This is converting an Observable to a Promise.

---

## Bridging to Promises

```javascript
var source = Rx.Observable.range(0, 3)
  .flatMap(function (x) { return Promise.resolve(x * x) })

var subscription = source.subscribe(
  function (x) { console.log('onNext: %s', x) },
  function (e) { console.log('onError: %s', e) },
  function () { console.log('onCompleted') })

// => onNext: 0
// => onNext: 1
// => onNext: 4
// => onCompleted
```

^This is flattening the stream of one much like resolving a Promise with a Promise does.

---

## Bridging to React

[RxReact](https://github.com/fdecampredon/rx-react)

^React is my new favorite UI framework since it's so simple.
While Rx and React have nothing to do with each other, RxReact brings them together nicely.

---

## Bridging to React

```javascript
var RxReact = require('rx-react')
var Rx = require('rx')

class MyComponent extends RxReact.Component {
  getStateStream() {
    return Rx.Observable.interval(1000).map(function (interval) {
      return {
        secondsElapsed: interval
      }
    })
  }

  render() {
    var secondsElapsed = this.state ? this.state.secondsElapsed : 0
    return (
      <div>Seconds Elapsed: {secondsElapsed}</div>
    )
  }
}
```

^Step 1: we need to stream data into the component.

---

## Bridging to React

```javascript
var FuncSubject = require('rx-react').FuncSubject
var React = require('react')
var Rx = require('rx')


var Button = React.createClass({
  componentWillMount: function () {
    this.buttonClicked = FuncSubject.create()

    this.buttonClicked.subscribe(function (event) {
      alert('button clicked')
    })
  },
  render: function() {
    return <button onClick={this.buttonClicked} />
  }
})
```

^Step 2: we need to stream data out.

---

## Into the Troposphere

 - Schedulers
 - Testing and Debugging
 - Backpressure
 - Dozens of operators

^There's a lot more to Rx that I just don't have time to cover.

^Schedulers control when data moves though the streams.
This is actually how testing is enabled.
Set the data in the stream; build your pipelines; process the next tick.

^Backpressure allows you to buffer or window events.
Use this when you can't keep up with each individual event.
Think of the latency when making network requests.

^There are many, many operators—at least as many as Underscore or lo-dash, but
Rx has async built in.

---

## Summary

 - Rx is huge.
 - But you can start small.
 - Once you know it, you can use it almost anywhere.

