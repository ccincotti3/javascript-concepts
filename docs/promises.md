# Continuing Async - HTTP Requests, and Promises, Microtask Queue

Before continuing, let's think more about the APIs provided by our runtime environments, and what rely on this callback system just to really hammer this idea home:

- Timers
- HTTP requests that return new data (XHR, fetch)
- User Interactions with DOM (clicks, form submissions etc)
- Read/Write to Filesystem (Node)
- Read/Write to Database (Node)

MDN offers a nice list of WebAPI's [here](https://developer.mozilla.org/en-US/docs/Web/API)

So as you can see, a lot of the useful things that we might want to do in Javascript rely on this asynchronous paradigm, so it's important to fully understand this in the case where bugs that arise.

We've seen the case of `setTimeout`, and that provided the foundation to understanding how asynchronous Javascript works. But what about HTTP Requests in the browser. 

First, we'll chat about `XMLHttpRequest` s in the browser, and how they work, and then we'll build off that information by moving into `Promises`, and how they're specifically handled. 

## XHR - XMLHttpRequest
The beauty of the _browser feature_ `XHR` is that if you understand how it functions, you can understand `fetch`, `axios`, `AJAX` etc. All of these libraries/functions leverage the built in browser mechanism to make these requests. Thus, it is crucial to understand this, as it will open the door to understanding `fetch` and its use of the `Promise` paradigm.

Luckily for us, `XHR` operates very similarly to `Timer` in terms of carrying out its asynchronous functionality, thus most of this will serve as a refresher for the previous section.

The main difference of course is that this time, we are telling the browser to do retrieve data from another server somewhere else on the world wide web. 

So let's get right into it:

```javascript
const oReq = new XMLHttpRequest();

function logData() {
	console.log(oReq.responseText)
}

oReq.addEventListener("load", logData);
oReq.open("GET", "http://www.example.org/example.txt");
oReq.send();
```

If you've never used the `XMLHttpRequest` object, that's not a big deal. All that's important here is that we're using javascript to spin up this feature in the web browser. Just imagine that we're building what you would normally pass into a `fetch` call.

What's important here is the line `oReq.addEventListener("load", logData);`  because it's here where we register our callback. 

So what's happening under the hood. So, high level we're telling the browser to make an HTTP request to fetch some data, and when that data comes back, call our `logData` function. 

 1. Once the request returns data, the browser attaches this return data directly to the `xhr` object through the `responseText` property, and `xhr.onload` is added to the message queue to be called once the call stack is empty.

From there, the steps are just like `setTimeout`

2. Once the call stack is empty, the event loop adds the `xhr.onload` callback (which is our `logData` function) to the call stack, to be executed by the Javascript engine. 
3. The code is executed.

Yeah, so not much new processes going on here. This was more of a review than anything. Moving forward, this opens the door to talk about `fetch` and `Promises`!

## Promises

[Let's start by looking at the definition provided by MDN:](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 

**A quick aside**: You can use the MDN docs to better understand what's Javascript, and what's a WebAPI. What is `fetch` under? [You can check here.](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)  _Hint: It's not Javascript_

If you clicked the link, did you notice that Promises are under the `Javascript` breadcrumb? That means that Promises are a Javascript feature, and not something provided by our runtime environment. Let's remember this for later.

Now, back to the definition...

> A **Promise** is a proxy for a value not necessarily known when the promise is created. It allows you to associate handlers with an asynchronous action's eventual success value or failure reason. This lets asynchronous methods return values like synchronous methods: instead of immediately returning the final value, the asynchronous method returns a _promise_ to supply the value at some point in the future.

So if we read the definition of Promises above, the sentence that sticks out to me is:

> This lets asynchronous methods return values like synchronous methods: instead of immediately returning the final value, the asynchronous method returns a _promise_ to supply the value at some point in the future.

Which means that we can _immediately_ return a Promise upon executing the function, and continue down our thread of execution. The `Promise` object will act as a placeholder for the data that we are expecting to eventually have.

So we can just append our `.then`'s and `.catch`, etc and everything will work as planned....

...but that begs the question, how does it all work? It was relatively easy to reason around our past asynchronous examples since our callback functions were expressly passed in to either the `XHR`, or `setTimeout`, but now we have an entirely new paradigm to wrap our heads around.

Well let's take a look at what we can expect to find on an instance of `Promise`

According to the ECMA spec, we can find the following properties on a `Promise` object:

- [[PromiseState]] - `pending`, `fulfilled`, or `rejected`
- [[PromiseResult]] - the returned data that we requested
- [[PromiseFulfillReactions]] - chain of functions to call after fulfillment, aka the  `.then`'s
- [[PromiseRejectReactions]] - the functions to call in case of rejection (aka `.catch`'s)
- [[PromiseIsHandled]] - boolean indicating that there is a rejection handler. (If there isn't you might see a warning in the console -> `UnhandledPromiseRejectionWarning`).

For now, all that we're concerned about is the relationship between `[[PromiseState]]`, `[[PromiseResult]]`, and `[[PromiseFulfillReactions]]`. 

So let's use a typical `fetch` call:
```js
const myFetchPromise = fetch("mywebsite.com/id/1")
					   .then(resp => console.log("Success: ", resp)
```

So `myFetchPromise` will first start out as something like this:
```
Promise {
	[[PromiseState]]: "pending",
	[[PromiseResult]]: undefined,
	[[PromiseFulfillReactions]]: [resp => console.log("Success: ", resp]
}
```

When the `PromiseState` is `pending`, the `PromiseResult` is `undefined`, because we're still awaiting the data to return. This is important because it shows that this `Promise` instance has been created before the return of our data from the `fetch` call.

Then the data comes back...

Now the `PromiseState` is `fulfilled`, we can find the data we requested in the `[[PromiseResult]]`, which then kicks off the promise chain found in `[[PromiseFulfillReactions]]`. The `PromiseResult` will be passed to the first function in the `PromiseFulfillReactions` chain. As a result, we now can operate on the data.

But how does this all work in the browser? If we consider something like `fetch`, which returns a `Promise` object upon invocation, now we have a weird combination where a browser feature like `fetch` returns to us a Javascript `Promise` object and somehow the browser is able to resolve the functions all the way down the promise chain.

Given these questions, let's hop right into understanding `fetch` to provide some insights.

## Fetch and the Job Queue

`fetch` is a great way to really tie in both Promises and the XHR request. Let's get right to it with some code.

```javascript
function log(data) {
	if(data) {
		console.log("Success")
		return "I'm a new promise result"
     }
}

const myPromise = fetch("https://www.google.com")
				  .then(html => log(html))
```

So here, we have `myPromise` which is a `fetch` call with a `.then` chained onto it to log whatever that response is (should be HTML).

When we call `fetch`, two things are happening: 

 1. It first returns to use a `Promise` object immediately, thus making `fetch` a non-blocking call. This happens on the Javascript side, of course.
 2. It also kicks off an `XHR` (XmlHttpRequest) in the browser! We give the browser all the information it needs to know by using  `fetch` to make an `XHR` request. 

It's clear then that `fetch` is really just a _facade_ in the sense that it is more or less syntactic sugar to take advantage of the nice `Promise` paradigm.... as well as avoiding the clunkiness of directly using the `XHR` API.

The code continues to be executed, and the browser is now responsible for returning some data to us, fulfilling the `Promise`, and then executing the functions chained onto the `Promise` with `.then` and `.catch`, etc. 

How does it resolve the `Promise`?

After it gets the data that we asked for, it assigns that data to the `myPromise`'s `[[PromiseResult]]` field, which simultaneously marks it as `fulfilled`. Now, the `[[PromiseFulfillReactions]]` will be called in order that they're defined in the `.then` chain. 

But when? and how? Let's review quickly: we saw in the first section of our asynchronous adventure that the _Message Queue_ exists which queues our callback functions to be added to the call stack to be executed once the call stack is available to do so.

...**That's not happening here**. Well it kind of is, but not quite. The _Message Queue_ is not being utilized in adding `Promise` chain functions to the call stack. In actuality, it's the **Microtask Queue** (sometimes referred to as the _Job Queue_.

So, back to where we were. The browser adds the first function from the  `[[PromiseFulfillReactions]]` list, and adds it to the Microtask Queue. The Event Loop then adds these functions to the call stack when it is available.

But what why is the browser using the Microtask Queue, _and what really is it?_ Let's find out.

## The Microtask Queue

So as we saw, the deferred callbacks from `Promise` objects are sent to the _Microstack Queue_ instead of the Message Queue. So what is a **microtask** ?

MDN defines it as:
> A **microtask** is a short function which is executed after the function or program which created it exits _and_ only if the JavaScript execution stack is empty, but before returning control to the event loop being used by the user agent.

But, can't this definition be applied to `Timer` callbacks? I asked this question myself, and MDN defines the difference between the two with the following: 

> At first the difference between microtasks and tasks seems minor. And they are similar; both are made up of JavaScript code which gets placed on a queue and run at an appropriate time. However, whereas the event loop runs only the tasks present on the queue when the iteration began, one after another, it handles the microtask queue very differently.

So the main difference lies in how the queues are called. 

Before diving into the differences - let's first review _what's the same_ for the _Message Queue_ and _The Microtask Queue_:
-   The queue is first-in-first-out: tasks enqueued first are run first.
-   Execution of a task is initiated only when nothing else is running.

No surprises here. So, knowing this, what are the differences?

As it turns out, _The Microtask Queue_ **gets priority over** the  _Message Queue_. On top of this, _The Microstack Queue_ must empty before anything on the _Message Queue_ will be added to the call stack.

So, if I have a microtask that adds 100 more microtasks to the _Microtask Queue_, then those 100 microtasks will be executed before, say, a `setTimeout` callback that's been sitting on the _Message Queue_ for a very long time, just waiting to be executed. 

Or, in terms of `Promise`s, if I have a promise chain of 100 `.then` call backs, they'll all be given priority to enter the call stack before anything else in the _Message Queue_.

**In summary** - the order is for execution is the following: regular code, then promise handling, then everything else, like other callbacks from DOM events, `Timers` etc.

## Additional Reading

 - [Using microtasks in JavaScript with queueMicrotask()](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)
 - [MDN Docs for Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
 - [Microtasks and event loop](https://tr.javascript.info/microtask-queue)
 - [MDN Docs for XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)