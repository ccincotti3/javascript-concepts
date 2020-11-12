# The Call Stack, Sync and Async
_Additional keywords_: Timers, promises, event queue 

For web developers who write in Javascript, there's a blurred line between Javascript code that is synchronous and the API's that are given to us by runtime environments (the browser, NodeJS, etc) that gives our code a more asynchronous nature.

This section will start from the very beginning of the story, by starting with the synchronous base behavior of Javascript, and we will see that these _extra features_ provided by the runtime environments itself actually affords us all the asynchronous behaviors that we know and love. **Spoiler alert**: Javascript alone has no concept of `fetch` or `setTimeout`. 

## Synchronous Javascript

### A word on the runtime environment and runtime engine

I think an entire section should be dedicated to these two, but just know that Javascript is executed by the runtime engine (V8 engine in probably all the cases that you've ever executed JS.) The runtime environment is everything that is needed to run your JS (compiler, even the engine, etc). Think NodeJS or a browser. 

Just placing this here so as to avoid any confusion if you choose to skip around!

### The Call stack
Let us revisit our code:
```javascript
console.log("Before Function")
function addThree(num) {
	const newValue = num + 3
	console.log("In Function")
	return newValue
}
console.log("After Function")
const myAnswer = addThree(1)
```

Returns:
```
Before Function
After Function
In Function
```

We sort've touched on this while learning about scope, but let's review. First, we define the function `addThree` . _Then_ once `addThree` is _invoked_ we will then step into the `addThree` function and step through its execution thread (provided by the newly formed execution context) line by line, until we hit the keyword `return`.

What's important to realize here is that as we stated, the code is executed line by line. That is to say - Javascript is executed **synchronously** and that every function invocation is considered **blocking**. If we invoke a function that takes 5 seconds to run to completion, then we cannot continue to the next line in the thread until that function returns.

How does the Javascript engine keep track of what line to execute, and when? What about the scope that it's in? How does the engine know where to go after it returns out of a function?

Through something called **The Call stack** .

We can think of the call stack as just as the name implies. A stack, which is a data structure that implements Last In First Out (LIFO). The last item in, is the first item that is removed.

For further clarification of this, imagine a stack as a stack...of plates. We cannot remove plates from the middle or bottom at will without of course making a big mess. We must remove the plates from the top first! Thus, Last In First Out... or last plate, first out!

In more technical terms, the last function that we call will be added to the 'top' of our stack, or **pushed** to the call stack. The tricky part here is that all other functions within the stack cannot be completed until the function's on top of it do first (remember the plates!). Once a function returns, it is removed, or **popped** from the call stack, which allows the call stack to go back to the scope where that recently-popped-from-the-stack function was originally was called. 

Ok, so now onto some code to help visualize this:

```javascript
function A(){
  console.log("Up - A - I'm at the top!");
  console.log("Down - A - Going back down");
}

function B(){
  console.log("Up - B")
  A();
  console.log("Down - B");
}

function C(){
  console.log("Up - C")
  B();
  console.log("Down - C");

}
console.log("Start - global")
C();
console.log("Done - global")
```

This will return 
```javascript
Start - global
Up - C
Up - B
Up - A - I'm at the top!
Down - A - Going back down
Down - B
Down - C
Done - global
```

Interesting, so if we study these console statements, we'll see that we bubble up to `A`, and then bubble back down after `A` complete's it's execution. So all the other function calls in the stack are 'blocked' until the ones above it return and finish their execution.

Let's look at a visual to better understand what just happened:
~~ Add file ~~

So as we can see, we push our function calls to the top of the stack, then after A is executed, B is unblocked and is able to complete, and so on. 

So Javascript knows where it is in the code during any moment during code execution based on what is at the top of the call stack! It can then find its way back down to the global thread by following the call stack, well, back down the stack.

An additional important concept to understand is that once we finish execution **the call stack becomes empty**. This will become more important when we talk about The Message Queue in the asynchronous section.

Wrapping up - this is great, but as web developers, we use asynchronous functions daily whether we're fetching data from API's, setting timeouts, etc. Where does this functionality come from? How does Javascript, which is synchronous in nature, handle this? Does it handle it at all? We'll see.

## Asynchronous Javascript

As we said before, javascript is synchronous and single threaded. That's true, and will remain true... until it doesn't. So how does it gain its asynchronous nature? By using _additional features_ provided by our favorite runtime environments to unlock the capabilities of **callbacks** and **promises** which unblock the thread!

### A quick word on WebAPIs and other functions.
This could also have maybe a section on its own. But it's worth saying that Javascript alone is not so full featured as we might have thought.

`setTimeout` and `fetch` are great examples of functions that are _not_ defined in the Javascript spec, but are provided by the runtime environment (browser, NodeJS, etc). 

There is a difference between what environments provide us. For example, the browser provides us functions commonly referred to as the **WebAPI** used to handle our web related needs such as HTTP requests, or DOM manipulation. NodeJS, as you can expect, does not need to worry about something like DOM manipulation. So it does not provide these features. However, it provides API's to read and manipulate files, while on the other hand, the WebAPI does not.

The moral of the story is - different environments provide different API's, and that a lot of our favorite API's as web engineers are _not_ built into plain old Javascript as we might have expected them to be. 

### The Event Loop, and the Message Queue (callback queue)

Ok, so we know that there is the call stack that keeps track of what is being executed at a given time. 

But what happens when we use `setTimeout` _in the browser using a WebAPI_? Let's take a look at an example.

```javascript
console.log("hello")
function question() {
	console.log("hows it going?")
	return null
}
setTimeout(question, 1000);
console.log("bye")
```

Which will return:
```
hello
bye
hows it going?
```

Does this make sense?

Doesn't this fly in the face of everything that we learned about in terms of Javascript's synchronous nature?. Namely, why was `console.log("bye")`  able to run before the result of calling `question` in `setTimeout` was returned? 

As it turns out, `setTimeout` was able to defer the invocation of `question` and allowed the execution of code to continue line by line, _thus unblocking thread_, and therefore the call stack.

But how? Well, `setTimeout` is not in the Javascript spec as one might think. We use it in our javascript projects, [but it is a feature provided by the runtime environment](https://nodejs.org/api/timers.html)  that instantiates a `Timer` class, which by the definition from the NodeJS docs:

> The  `timer`  module exposes a global API for scheduling functions to be called at some future period of time. Because the timer functions are globals, there is no need to call  `require('timers')`  to use the API.

> The timer functions within Node.js implement a similar API as the timers API provided by Web Browsers but use a different internal implementation that is built around the Node.js  Event Loop.

Two things jump out at me in this definition:

 1. An API that is not apart of Javascript itself is giving us the ability to schedule functions in the future.
 2. What is an Event Loop? (Spoiler, we're getting to that very soon).

Interesting. So it seems that after a specific amount of time, our `timer` will return our function `question` and it'll be executed.

_... well not quite._

So yes, it will be returned back to the call stack, _eventually_. But just sending it back to the call stack would not work. Let's see why:

```javascript
console.log("hello")
function blockingFunctionFor100Seconds() {
	/* Imagine a loop here that takes 
	** 100 seconds to complete 
	*/
}
function question() {
	console.log("hows it going?")
	return null
}
setTimeout(question, 1000); // 1 second timer
blockingFunctionFor100Seconds(); // 100 second blocking 
console.log("bye")
```

If we run the above code, and we set a `Timer` with a duration of `1000ms` (or `1s`), and immediately after, we hit our `blockFunctionFor100Seconds`, will `question` be executed while our blocking function is still being executed? The answer is no, absolutely not. 

So then, you might be thinking, OK after our blocking function completes, then we will execute `question`. Yes, technically, _but not immediately after_. In fact, we will see this again:

```
hello
bye
hows it going?
```

Alright let's just be blunt about it now: The deferred function call will  be executed **once there is nothing left on the call stack**.

As it turns out, the call stack is not the only data structure that comes into play. There's a second data structure that exists where our runtime environment will _queue_ deferred callback functions once they are ready to be re-entered back into the call stack. This second queue is called **The Message Queue** or sometimes called the Callback Queue or The Task Queue.

If we consider the behavior of a queue, (First In First Out), we can assume the behavior of the message queue. The first item added to it will be the first to be added to the call stack to be executed _once the call stack is completely empty_. 

If this is the case, then to really wrap your head around this concept, it is crucial to understand that the time in `ms` that we pass to `setTimeout` is the _minimum_ required of time that must pass before the Javascript engine will execute the callback that was passed in. There is no guarantee that the call stack will be empty after the `timer` has completed.

Now that we understand The Message Queue, we can now understand **The Event Loop.**

The Event Loop's job is to constantly check whether the call stack is empty. If it is, it will _dequeue_ or remove the item in the front of the queue and add it to the call stack, to be executed. That's it - it's just a loop that constantly checks the call stack!

Here is a visualization to better explain this concept.

Find an image.

Alright, we learned a ton in this section. Let this simmer, and we'll dig even deeper into this concept of asynchronous Javascript with Promises, and async/await.

## Additional Reading

 - [When is Javascript synchronous?](https://stackoverflow.com/questions/2035645/when-is-javascript-synchronous)
 - [The JavaScript runtime environment](http://dolszewski.com/javascript/javascript-runtime-environment/#:~:text=JavaScript%20engine%20vs%20runtime%20environment&text=The%20engine%20translates%20scripts%20at,script%20that%20references%20these%20libraries.)
 - [How does NodeJS work(Beginner to Advanced)? — Event Loop + V8 Engine + libuv threadpool](https://chaudharypulkit93.medium.com/how-does-nodejs-work-beginner-to-advanced-event-loop-v8-engine-libuv-threadpool-bbe9b41b5bdd)
 - [What does it mean by Javascript is single threaded language](https://medium.com/swlh/what-does-it-mean-by-javascript-is-single-threaded-language-f4130645d8a9)
 - [Javascript — single threaded, non-blocking, asynchronous, concurrent language](https://theflyingmantis.medium.com/javascript-single-threaded-non-blocking-asynchronous-concurrent-language-ffae97c57bef)
 - [Asynchronous Javascript Part 3: The Callback Queue](https://levelup.gitconnected.com/asynchronous-javascript-part-3-85390632dd1a)
 - [The Event Loop](https://flaviocopes.com/javascript-event-loop/)
 - [In-Depth Introduction to Call Stack in JavaScript.](https://medium.com/swlh/in-depth-introduction-to-call-stack-in-javascript-a07b8513bcc3)