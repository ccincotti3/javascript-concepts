# Chapter 1: Variable Keywords and Hoisting

The differences between the keywords `var`, `let` and `const` are rather inconsequential for most developers of modern Javascript. Most dev's know the golden rule to stick to `const` unless a variable is expected to be reassigned in the future (like in an if condition, or a for-loop). However, there are time's where dev's are called to work in legacy code, or they stumble across an old StackExchange answer, where `var` is employed.

At first use of `var` it's usually seen as to declare a super variable in some respects. Like with the use of `let`, we can change a variable it's later if we want to. We don't have to think, why not just use `var`?

So I propose that we start with `let` vs `var`.

Let's start here by defining two variables in the global scope with these two keywords `let` and `var`:
```javascript
let myLet = "Apple"
var myVar = "Pie"

console.log({ myLet, myVar })
// { myLet: 'Apple', myVar: 'Pie' }
```
_**Pro-tip** - logging variables as an object increases readability, because key-value pairs are logged to the console._

OK, no surprises here. We defined two variable's in the global scope, and we logged them to the console.

Now let's create a function, and really get to the meat and potatoes. Let's show a key difference by logging these two variable's to the console _before_ they are defined.

```javascript
function myFunction() {
	console.log({myVar})
	console.log({myLet})
	let myLet = "Apple"
	var myVar = "Pie"
}
myFunction();
```

Running this yields: 
```
{ myVar: undefined }
Thrown:
ReferenceError: myLet is not defined
```

What? `myVar` is undefined, and `myLet` returns a `ReferenceError` that states that myLet has yet to be defined. 

This brings us to a major difference between these two. However, before we can really understand this difference, it's important to understand how Javascript is built and ran, and from this we will understand the concept of **hoisting**. 

We will explore the topics more in depth when we discuss Closures/Scopes and how execution of our code really works, but what's important for this section is to understand that when we're **interpreting** code, there are 2 phases. (This is a generalization, each JS engine has many steps depending on what optimizations it opts to take, but simplifying it down to two steps is OK.)

Whether it be

 - The global context
 - a function
 - an eval statement

We first start with **The Creation Phase** and then following that, **The Execution Phase**.

At a super high level (we'll revisit this concept later):

 1. **The Creation Phase** is when the JS interpreter stores allocates space for variable and function declarations in memory.
 2. **The Execution Phase** is when the JS interpreter starts executing the code line by line. Therefore, our variables will be assigned the values that we defined. So `myVar` will actual be assigned `Pie` for example.

So using the knowledge that we are know equipped with, we can deduce that `myVar` was defined a default value of `undefined` during the Creation Phase (before it was actually assigned a value during the Execution Phase). So, `myVar` had a value of `undefined` even before the assignment of `myVar` was executed. 

But that doesn't explain `myLet`, well, can we be sure that the variable was hoisted within the function? Let's see another example to truly test it. This time, let's define a variable `let` myLet, and _another_ within the function. This is possible due to scope (again we'll cover that later).

```javascript
let myLet = "Pumpkin"
function myFunction() {
	console.log({myLet})
	let myLet = "Apple"
}
myFunction();
```

Which returns -
```
Thrown:
ReferenceError: myLet is not defined
    at myFunction (repl:2:15)
```
Wait, a minute... _what_? 

We just hit our first difference between the two! The JS interpreter does not define `let` with _undefined_ in the 'Creation Phase' but instead it is considered not 'initiated' at all. But, and it's important to note, **it is still hoisted** within its scope and thus assigned space in memory during the Creation Phase. We see this by overwriting the `myLet` from the global scope in memory. This state where the variable is allocated memory, but is inaccessible is named the **Temporal Dead Zone**. 

We can drill down these topics further, but for the sake of staying on track, let's move on.

So the other key difference that we'll see is expressed by the following code:

```javascript
var myVar = "FirstVar"
let myLet = "FirstLet"
function myFunction() {
	console.log({myVar})
	console.log({myLet})
	{
		var myVar = "SecondVar"
		let myLet = "SecondLet"
	}
}
myFunction();
```

Returns:
```
{ myVar: undefined }
{ myLet: 'FirstLet' }
```

Uh what? 

So `myVar` is returns `undefined` and `myLet`  is `FirstLet`.

Putting our thinking caps on, the last time we saw `myVar` as `undefined`, we understood that it had to do something with the 
Creation Phase' and the hoisting of the variable... OK. So I think we can convince ourselves that that same phenomenon happened here.

But that doesn't explain `myLet`. It looks like it inherited the `let` keyword from the global scope, and did not change at all when it was reassigned within that block. That's worth repeating one more time: it did not change at all when it was reassigned. Within. That. Block. 

We just hit our second difference between the keywords `var` and `let` which is: `var` is **function-scoped** and `let` is **block-scoped**. Meaning, the `var` defined variable is hoisted within the nearest function definition of which it resides, while `let` does not. `let` is hoisted within to the nearest _block_ (think of a block as anything within brackets - conditionals, loops, even functions as we saw in a past example, etc).

So in summary the two main differences between `let` (as well as `const`) and `var` are:

 1. How each are initialized in the 'Creation Phase'
	 - `var` is initialized with `undefined`
	 - `let`/`const` are initialized as **Temporal Dead Zones** and thus are considered not accessible until they are defined in the Execution Phase.
 2. How each are scoped
	 - `var` is function-scoped.
	 - `let/const` are block-scoped.

Next, we'll take a deeper dive into scope and closures!