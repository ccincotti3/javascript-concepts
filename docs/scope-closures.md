# Scope, Execution, and Closures
In this section we'll be exploring how code is executed in Javascript. Through this, we'll understand the foundation of the language, which will enable us to better understand the topics that follow.

Let us start at the _the very beginning of this story_ with the following code:
```javascript
const myConst = "Cool Const"
console.log("Before function definition")
function addThree(num) {
	console.log("In the function")
	const newValue = num + 3
	return newValue
}
console.log("After function definition")
const newNumber = addThree(4)
console.log("End of File")
```

When we run this, we will receive:
```
Before function definition
After function definition
In the function
End of File
```

From this code we learn that the execution of Javascript happens **line by line**. We start at the top and work our way down. This is because javascript's execution is single threaded and synchronous in nature. 

However, we do _not_ step into the function until it is invoked with the line `const newNumber = addThree(4)`, and we see this represented in the console.

Knowing this will set us up to now understand scope!

## Scope (global, function, block)

Before I diverge on my own thoughts on scope, [let's look at the actual definition from MDN](https://developer.mozilla.org/en-US/docs/Glossary/Scope): 

> The current context of execution. The context in which values and
> **expressions** are "visible" or can be referenced. If a variable or other expression is not "in the current scope," then it is unavailable
> for use. Scopes can also be layered in a hierarchy, so that child
> scopes have access to parent scopes, but not vice versa 
> _MDN_

Adding to that definition: I tend to think of scope as _what is accessible in memory_, and what's not, or more simply what I can access versus what I can not. 

## Global, function, and block scope

So let's step through this code below to gain a better understanding of what's going on in memory:

```javascript
const myConst = "Cool Const"
function addThree(num) {
	const newValue = num + 3
	return newValue
}
```
Here's a table of what we would expect to see in the **global scope**, and thus the **global memory** after running this code:
|Line|Memory|
|--|--|
|const myConst = "Cool Const" |myConst: "Cool Const"|
|function addThree(num){...}|addThree: function(){...}|

So before we continue, it's important to realize that our keywords `const`, `let`, `var`, `function` etc are telling the interpreter 'hey store this in memory'. How things are actually being stored (value vs reference) is another story, we will cover that in another section. But it's not important right now.

But what about `newValue`, or even `num`? Well, we don't have access to that in the _global scope_.  Those are **function scoped**, and thus exist within the _local memory_ of the function. We'll revisit `addThree` soon... let's first see the last scope type:

```javascript
const myConst = "Cool Const"
if(true) {
	const blockConst = "I am in a block"
}
```
Here's a table of what we would expect to see in global scope after running this code:
|Line|Memory|
|--|--|
|const myConst = "Cool Const" |myConst: "Cool Const"|

Ah, so we cannot access `blockConst` in the global scope, because this is variable is actually **block scoped**. 

OK, so now that we have defined the three different scopes (global, function, and block) we can now understand what happens when we invoke `addThree` .

## The Execution Context

So what happens when we invoke  `addThree`? This is important, so much so, I'm going to give it it's own line:

We create an **execution context** .

What is that exactly? I'll spell it out shortly, but it's rather intuitive if we consider what we already know which is:

 1. We enter the function once we invoke it. We've seen this in the introduction.
 2. A function has its own scope, or its own local memory.

So let's look at the code again, but this time tie together global and local scope:
```javascript
function addThree(num) {
	const newValue = num + 3
	return newValue
}
const answer = addThree(1)
```

So let's first look at the global scope:
|Line|Memory|
|--|--|
|function addThree(num){...}|addThree: f()|
|answer|?|

What is `answer`, well technically it's not defined (as opposed to `undefined`, you will receive a `ReferenceError`). What is important is that it's now waiting for the return value of the invocation of `addThree`. So, the Javascript engine now needs to call `addThree` - which as a result _causes the creation of a new execution context_, or thread of execution, for `addThree`.

Now, just what is an **execution context**? It's a new thread of execution where we step line by line through the function itself. The thread of execution has it's own locally memory, or - it's own memory 'scoped' to it (function-scope).

So, step through `addThree` and take a look at the local memory (scope).
|Line|Memory|
|--|--|
|num (parameter)|num: 1|
|const newValue = num + 3|newValue: 4|

Then we return the _value_ of  `newValue`, which as we see in the local memory is 4, _to_ the global scope, and store it in the variable `answer`. **The execution context is now garbage collected**. Everything about that execution context is gone now. We now return to the global scope (which you can consider a global execution context) and continue executing the code line by line until we hit the end of the file. 

 So, we would expect the memory in the global scope to resemble: 
|Line|Memory|
|--|--|
|function addThree(num){...}|addThree: f()|
|answer|answer: 4|

**A quick aside:** The process of managing execution contexts has to do with the Call Stack, which we will talk about in a later lesson.

Let me introduce a teaserb before we visit the next concept. Let's finally revisit our original code:
```javascript
const myConst = "Cool Const"
function addThree(num) {
	const newValue = num + 3
	console.log(myConst)
	return newValue
}
const newNumber = addThree(4)
```

If we run this, the following will be logged in the console:
`"Cool Const"` . 

So we have access to `myConst` within the function scope, which means that _from the function scope, we can access the global scope_. Meaning, we are able to _close_ around an outer scope's memory, through the definition of a function and thus have access to it. We call this combination of a function's memory and it's surrounding memory **closure.**.

## Closures

[MDN defines a closure as](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures):
> A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time.

Think of scope as a [Matryoshka doll, or more commonly known as a Russian doll](https://www.google.com/search?q=matryoshka%20doll&source=lnms&tbm=isch&sa=X&ved=2ahUKEwiF6KGqjezsAhUyy4UKHU9-ApEQ_AUoAXoECCkQAw&biw=1792&bih=1041). The inner dolls live within the larger doll. In the context of scopes - the inner scopes have access to the outer scopes. 

So, if we have our global scope, theoretically every scope that we define with the global scope will be able to access whatever has been defined within the global scope. 

Let's revisit our previous example:

```javascript
const myConst = "Cool Const"
function addThree(num) {
	const newValue = num + 3
	console.log(myConst)
	return newValue
}
const newNumber = addThree(4)
```

When we invoke `addThree`, we create an execution context. So, let's take a look at what's in there:
|Line|Memory|
|--|--|
|num (parameter)|num: 1|
|const newValue = num + 3|newValue: 4|
||myConst: "Cool Const"|

Looks like we get `myConst` for free! This is an important topic as we just learned how we can share memory amongst our functions without explicitly passing data. And, we'll also see soon something _way more powerful_. But for now, let's see what's happening under the hood.

As it turns out - **inner scopes have a reference to their parent scope** through a hidden property of name `[[Environment]]`, that keeps the reference to the surrounding environment where the function was defined.

So, returning to:
```javascript
const myConst = "Cool Const"
function addThree(num) {
	const newValue = num + 3
	console.log(myConst)
	return newValue
}
const myFunc = addThree
```
We can deduce that` myFunc.[[Environment]]` has a reference to `myConst`. This hidden property is not accessible to the user, but the engine uses it to find `myConst`. If it can't find it in the immediate surrounding environment, it will recursively check the `[[Environment]]` field in the encapsulating outer scopes. If it is not there, it will return a `ReferenceError`.

But what does this imply? It implies that parent scopes _are not deleted_ since a child scope may need to access the data at any point during execution whenever and wherever. 

This can be annoying though, because now our inner functions can now manipulate the world around it!

```js
let i = 0
function tick() {
	return i++;
}
tick()
tick()

console.log(i) // 2
```

So you really need to be careful... but closures also come with great power.

### Why Closures are powerful
```javascript
let counter = 0
function tickCounter() {
	counter++
}
tickCounter()
tickCounter()
tickCounter()

console.log(counter)
```

`counter` will log out 3. This is because `tickCounter` has a reference to `counter` (by accessing the `[[Environment]]` property) and is able to iterate it by 1. 

So now consider: 
```javascript
function counterWrapper() {
	let counter = 0
	return function tickCounter() {
		counter++
		return counter;
	}
}
const myFunc = counterWrapper()
const mySecondFunc = counterWrapper()

let myFuncCounter = 0;
let mySecondFuncCounter = 0;

myFuncCounter = myFunc()
myFuncCounter = myFunc()
myFuncCounter = myFunc()

mySecondFuncCounter = mySecondFunc()

console.log({myFuncCounter, mySecondFuncCounter})
```

Running this code, we get `{ myFuncCounter: 3, mySecondFuncCounter: 1 }`. What did we just do? Well by invoking our `counterWrapper` function, we returned the function `tickCounter`, that closed around the surrounding scope _where the function was defined_. So now, both `myFunc` and `mySecondFunc` have their _own_ references to `counter`, and thus, as we've seen, can manipulate their own version of `counter` without affecting the other.

How did this happen? Well by storing the result of `counterWrapper` (which is the `tickCounter` function) into the variables `myFunc` and `mySecondFunc`, we created entirely new _references_ to `tickCounter` in memory, and with it, the memory that was closed around it. So now, `myFunc` and `mySecondFunc` have their own unique copies of the `counter` variable.

_This is powerful_. **We now have a way to shove data into a child function and have this data persist**. By using closures, we can create our own stateful functions!

## Summary

1. There are 3 types of scope - global, function, and block.
2. Javascript is synchronous and is executed line by line.
3. We execute the code within functions when they are called, and create an execution context.
4. Memory is scoped, but inner scopes can access outer scopes.
5. Function definitions close around their surrounding environment, and we call this combination of a function bundled together with references to it's outer scope a closure.

Now that we understand scope, and how execution works. The next step would be to better understand the Call Stack so we can understand just exactly how Javascript handles the invocation of many functions at once. 

## Additional Reading

- [https://developer.mozilla.org/en-US/docs/Glossary/Scope](https://developer.mozilla.org/en-US/docs/Glossary/Scope)
- [https://codeburst.io/explaining-value-vs-reference-in-javascript-647a975e12a0](https://codeburst.io/explaining-value-vs-reference-in-javascript-647a975e12a0)
- [https://dev.to/sandy8111112004/javascript-introduction-to-scope-function-scope-block-scope-d11#:~:text=A%20block%20scope%20is%20the,only%20within%20the%20corresponding%20block.](https://dev.to/sandy8111112004/javascript-introduction-to-scope-function-scope-block-scope-d11#:~:text=A%20block%20scope%20is%20the,only%20within%20the%20corresponding%20block.)
- [http://speakingjs.com/es5/ch16.html](http://speakingjs.com/es5/ch16.html)
- [https://javascript.info/closure](https://javascript.info/closure)
