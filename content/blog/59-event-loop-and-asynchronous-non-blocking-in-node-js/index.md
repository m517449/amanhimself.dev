---
slug: event-loop-and-asynchronous-non-blocking-in-node-js
date: 2018-06-02
title: 'Event Loop and Asynchronous Non-Blocking in Node.js'
categories: ['nodejs']
description: '---'
published: true
author: 'Aman Mittal'
banner:
---

## Introduction to Node.js Event Loop

Node.js is single threaded. It supports concurrency through paradigms of event and callbacks. Since it is single threaded, most APIs provided by Node.js core are asynchronous. They follow a non-blocking Input/Output or I/O. What is a non-blocking Input/Output you ask?

In a traditional Input/Output, when a request comes to a web server, it is assigned to a specific thread. For each concurrent connection, there is a new thread and the thread will continue to run until a response is sent for a particular request. This is a perfect example of Blocking I/O (Input/Output network operation) because when handling a particular request by a thread there will be some idle time when between operations are being performed such as retrieving a file, opening it, reading it, etc.

Each of these thread consumes memory and thus, a thread which runs for a longer period of time and also sits idely for a significant amount of time in between will definitely consume a lot of memory. This is one of the main reason that the core APIs in Node.js are built in non-blocking way and support asynchronous I/O.

Node.js uses V8 that is a runtime engine developed by Google for Chrome browser to run JavaScript. Basically, V8 is responsible to convert code written in JavaScript into machine level code. Different browsers have different runtime engines. It is the reason Node.js is fast and efficient.

Hence, we can say that Node.js is itself a runtime environment. It follows an observer pattern (also known as Reactor Pattern in other programming languages) which allows each incoming I/O request to be associated and handle within a function or a handler. This handler function is called a callback function.

At the time of running a Node.js environment, an Event Loop is initialized which handles the I/O operations by offloading them to an operating system's kernel. Different operating system use different kernels but their basic mechanism of handling an I/O is similar. These kernels are multi threaded and can handle execution of multiple operations in the background. Whenever an I/O operation completes, the Kernel notifies Node.js about it and callback handling that operation will complete its execution.

All of these operations are queued in a poll which is also known as Event Queue. Any of these operations may proceed to more operations and these new operations are then again added to the Event Queue. In summary, Event Loop will always be responsible for the execution of all the asynchronous callbacks registered for every event in the Event Queue. Other than I/O operations that are queued in the Event Queue or poll the other types of callbacks can also execute with the Event Loop. These other types are:

- Timers
- `process.nextTick()`

## Timer Functions

In JavaScript, a timer function will always have callback associated with it. This callback indicates the execution as early or defer it, in an Event Loop after the specified amount of time is passed. Each timer function accepts this value in milliseconds. When writing a Node.js module, these functions do not need to be imported via `require()`. All of these functions are available globally to us by Node.js core API similarly to the browser's JavaScript API. The behaviour of execution of these timer functions in Node.js differs that from the browser API. Let us take a look at them.

**setTimeout()**

This timer function is used to schedule the callback associated after described period of time in milliseconds. The first argument to this function is always a callback which can also be declared separately. The second argument of the timer function is the amount of milliseconds defined.

```javascript
setTimeout(() => {
  console.log(`Set Timeout executed`)
}, 1000)
```

In the above example, the callback associated will execute as close to 1000 milliseconds (or 1 second) as possible due to the call of `setTimeout()`. The Event Loop never guarantees that a `setTimeout()` function will execute exactly after the expected time. However, it does guarantees that it will not execute before the specified amount of time. This is because there might be other executing asynchronous/synchronous functions might pushed before the execution of a `setTimeout()` function.

**`setImmediate()`**

This timer function behaves differently and only execudes the code associated at the end of an Event Loop cycle. This code will execute after every I/O operations in the current event loop and before any timers scheduled for the next event loop.

```javascript
console.log('before immediate')

setImmediate(arg => {
  console.log(`executing immediate: ${arg}`)
}, 'during immediate')

console.log('after immediate')
```

The above snippet of code will output as:

```shell
before immediate
after immediate
executing immediate: during immediate
```

The first argument is again is the callback that assoicaites the code to be deferred. Any other optional arguments can be passed to the function when it is executed, like we we have done above in our example. There is no need to define a time limit to defer the execution of this function since it will always run at the end of an Event Loop cycle in any Node.js program.

**`setInterval()`**

Now suppose, there is a callback that you want to execute in your Node.js module a multiple times. In that case,`setInterval()` can be used. It again, takes a callback as the first arguement and runs infinite number of times. The delay between each iteration is defined in milliseconds. Otherwise, this timer function behave is written in a similar manner as of `setTimeout()`.

```javascript
function runMultipleTimes() {
  console.log('Executing after One Second')
}

setInterval(runMultipleTimes, 1000)
```

To stop a `setInterval()` function from executing infinite number of times, we can use another function that will perform stop its behaviour, `clearInterval()`.

**process.nextTick()**

Apart from all timer functions being global, there is another object that global and does need to be required in any Node.js module. It is called `process`. Discussing everything this object holds is vast discussion and is out of the scope of this article. However, we are interested in a particular method the API contains and it is called `process.nexTick()`.

This function, `process.nexTick()`, is used in applications to defer the execution of an asynchronous function until the next cycle of Event Loop. Each cycle of the Event Loop is called tick. `process.nextTick()` runs a callback function. Basically, it is used to manually defer a callback function.

It does not take the argument of defining a time limit as other timer functions. Another major difference between this function and other timer functions is that `process.nextTick()` is only available in Node.js while other are also available in JavaScript's browser API.

```javascript
function callback() {
  console.log('Processed in next iteration')
}

process.nextTick(callback)

console.log('Processed in the first iteration')
```

If you save the above program in `.js` file and try running it using `node index.js`, you will notice that the second `console.log` statement is printed before the first statement associated within the function `callback()` even their callback is defined and called before the last `console.log` statement. Console statements in JavaScript and in Node.js are sychronous. The following output:

```shell
Processed in the first iteration
Processed in next iteration
```

## Event Emitters

In Node.js events are assoiciated with operations in Event Loop. For example, a TCP network server emits a _connect_ event every time a new client connects, or a stream can emit a _data_ event every time a new chunk of data is available to read. These objects in Node.js are called _event emitters_.

These objects that emit events are defined as the instances of the class `events.EventEmitter`. These objects reveal an `eventEmitter.on()` function that allows one or more functions to be attached to named events emitted. When an `EventEmitter` object emits the event, the functions attached to that specific event are called synchronously.

```javascript
const events = require('events')
const eventEmitter = new events.EventEmitter()

eventEmitter.on('event', (a, b) => {
  const c = a + b
  console.log('result: ', c)
})

eventEmitter.emit('event', 1, 2)
```

Using `eventEmitter.emit()` we can execute a listener function with appropriate arguments. EventEmitter class allows a subscription based model to define callbacks also known as _pub/sub_ mechanism. Please note that all the listeners attached to a particular event object are all synchronous functions and execute in the order they are registered.

> [Originally published at javabeginnerstutorial.com](https://javabeginnerstutorial.com/node-js/event-loop-and-asynchronous-non-blocking-in-node-js/)
