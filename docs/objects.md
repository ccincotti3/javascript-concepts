# Objects, Prototypes, and Classes (OOP)

Object Oriented Programming is a paradigm for organizing code. In this section, we'll take a look into inheritance, the prototypal chain, the various keywords such as  `class` , `new` and `this`, and what it means to use these features within our code.

## Objects

Let's start at the bare minimum, an object.

```js
	const myPantry = {
		apples: 2,
		oranges: 3,
		incrementOrange: () => myPantry.oranges++,
	}

	myPantry.incrementOrange();
	console.log(myPantry)
	/**
		logs 
		{ 
		  apples: 2,
		  oranges: 4,
		  incrementOrange: [Function: incrementOrange]
		}
	*/
```

Here, we define `myPantry` which is an object consisting of various value types. Notably, we defined a function to the object named `incrementOrange` which is able to increment the `oranges` field on the same object. 

This is defined in programming terms as **encapsulation**. MDN defines this as
> Encapsulation is the packing of data and functions into one component (for example, a class) and then controlling access to that component to make a "blackbox" out of the object. Because of this, a user of that class only needs to know its interface (that is, the data and functions exposed outside the class), not the hidden implementation.

This is fundamentally important since this serves as a basic motivation as to why we would use OOP versus something like functional programming. We are able to define all relative data and functions inside one 'blackbox' component, which provides an API to the user while hiding the implementation.

So this is great and all, but is pretty limited. What if I have multiple pantries? What if I have 500 pantries? Now, I'll have 500 copies of the properties in memory. What if I want to share properties amongst my pantries? Manually creating objects will get cumbersome very quickly... so let's use Object.create to help out!

### Object.create() and Prototypal Inheritance

In short, `Object.create()` will create for us an empty object, but with a link to the object that we pass in as an argument, giving us access to all the fields defined on that argument. If this is confusing, an example might clear this up:

```js
const cat = {
  isCat: false,
  printIntroduction: function() {
    console.log(`My name is ${this.name}. Am I a cat? ${this.isCat}`);
  }
};

const bernice = Object.create(cat);
console.log(bernice) // {}

bernice.name = 'Bernie'; // "name" is a property set on "me", but not on "person"
bernice.isCat = true; // inherited properties can be overwritten

bernice.printIntroduction();
// expected output: "My name is Bernice. Am I a cat? true"
```
Check out the line where we log `bernice` to the console right after calling `Object.create(cat)`. We are returned which looks like an empty object `{}`. But, there's a property that you may or may not see depending on where you run this code. 

If I inspect this object in a console that's relatively detailed like within a browser (I'm using Edge), we'll see a property on the object named `__proto__` which is defined as:
```json
"__proto__": {
	"isCat": false,
	"printIntroduction": f(),
	"__proto__": Object
}
```

This `__proto__` field stores properties from the object that we passed into `Object.create`, which can be accessed by sequential instances that are created by this base object. We see this by being able to call `printIntroduction` without needing to explicitly define this on the `bernice` object. We essentially made a "cat factory" that creates cat objects!

But how does this actually work? 

As it turns out, `__proto__` is a _reference_ to the base object. Javascript is smart enough to know that if it doesn't find a property on the child object, then it will move up the **prototypal chain** (or follow the `__proto__` link) to the parent object and look for the property there. It will do this up until it can't go any further up the chain, and finally return `undefined`, or an error if you try to do, say, invoke `undefined` by accident.

This concept is known as **prototypal inheritance**. 

So I mentioned that Javascript will go up the prototypal chain until it can't any further. When does it reach the end? That depends on the type of data on which we are invoking methods, but eventually we'll hit the `Object.prototype`. All objects in javascript will default to having a `__proto__` reference to `Object.prototype`, either directly, or as we seen - through inheritance of base objects up the chain and finally ending at  `Object.prototype`. 

Let's see this in action:
```js
bernice.toString() // '[object Object]'
```
`[object Object]` is really just the string representation of an object in Javascript. But what's important is - where did `toString` come from? We did not add this to our `cat` function. 

`toString` actually lives in `Object.protoype`. After not finding `toString` in the `bernice.__proto__` object, Javascript's interpreter was smart enough to check the object referenced by `bernice.__proto__.__proto__`, which contains the function `toString`. 

The `Object.prototype` also has a `__proto__`, which points to `null` effectively signaling to the interpreter that the method was not found.

So we can assume that technically _everything in javascript is inheriting the `Object` type_, and thus everything is technically of type `Object` except `undefined`. Even `null` is of type `Object`, but that's a different story.

## The `this` keyword
Did you see how in our previous example - we defined the method `printIntroduction` as the following:
```js
printIntroduction: function() {
    console.log(`My name is ${this.name}. Am I a cat? ${this.isCat}`);
```

We were able to use a `this` keyword to reference the `bernice` object that we created using the "cat" factory (with `Object.create`). Where is `this` coming from?

If you checked out the `scope` lesson, you should be familiar with the concept of the _execution context_ and _local memory_ and how they are used during the invocation of a function. If not, I recommend checking it out. High level, we get `this` for free. The `this` keyword is stored in the local memory of the function upon invocation. Its value is the object of which we invoked a method upon. So, in the case of `bernice.printIntroduction`, the `bernice` object is referenced by `this`. The object to the left of the `.` (dot) will become our `this`.

There are quite a bit of caveats with `this` however, and it pays to be careful.

### `this` is not that

Defining `this` correctly can be tricky if you're not careful. Again, Javascript handles this automatically, so one just needs to be sure that it sets it correctly for you. 

```js
	const myPantry = {
		oranges: 3,
		firstIncrementOrange: () => this.oranges++,
		secondIncrementOrange: function() {this.oranges++},
		thirdIncrementOrange: function() {
			const innerFunction = () => this.oranges++
			innerFunction()
		},
		fourthIncrementOrange: function() {
			function innerFunction(){this.oranges++}
			innerFunction()
		},
	}
```
I've gone ahead and defined four increment functions to better understand how `this` is not always what we think it is. Let's look at these individually.

`firstIncrementOrange`  **will not work** as we expected. It will reference the global scope (`window` in the browser.) Why? **The arrow function inherits the outer scope's `this`**, and thus in this case is `window`.

`secondIncrementOrange` is correct, and will work as expected.

`thirdIncrementOrange` is correct, because as we seen, the arrow function inherits the correct `this` from the outer function.

`fourthIncrementOrange` is **not correct**, because the `this` is once again inherited from the global scope.

**Summary**: so it's clear that we must be explicit by defining a method with the  `function` keyword, or use an arrow function to inherit the correct `this` value. 

Therefore, we can deduce that all functions by default inherit a `this` value, which is the global scope, and that we must make sure that Javascript knows to use the correct `this` by following the rules above.

## `new` and `class` keywords
Now that we have a better understanding of objects, prototypes, inheritance, etc, what exactly happens when we call `new` and `class`? 

To be frank, these keywords are used to make our lives easier. We've already learned what's happening under the hood in the previous examples. These keywords do most of the heavy lifting for us.

```js
function CatCreator(name, color){
	this.name = name
	this.color = color
}

CatCreator.prototype.printIntroduction = function(){
	console.log(`My name is ${this.name}`);
}

const bernice = new CatCreator("Bernice", "gray")
bernice.printIntroduction();
// expected output: "My name is Bernice."
```

So what's different here right off the bat is that `CatCreator` is considered a `constructor` function. In the easiest of terms, a constructor function acts as a blueprint to create many instances of objects of the same "type". When we use `new` before the constructor function, we are returned back an instance of this type, in this case `bernice`.

What's interesting here is that the constructor function also comes with a `prototype` property (remember functions are both functions and objects in javascript!) that we can use to add shared functionality across our instances. This is where we're saving `printIntroduction`. I've added [an interesting read](https://www.thecodeship.com/web-development/methods-within-constructor-vs-prototype-in-javascript/) outlining the differences of saving methods in the constructor function itself, versus it's `prototype` object. Give it a look when you can!

Anyway, when we invoke `CatCreator` by using the `new` keyword, `this` is immediately given the value of an empty object `{}`, with a `__proto__` referencing the `CatCreator.protoype`. Properties (`name`, `color`) are then added to it. 

We are then returned an object with the properties as defined in the constructor, and a reference to the `CatCreator.prototype`. 

As you can see, this is similar to our previous example, where we used `Object.create` to get the same outcome. As I've mentioned, this is just another possible - and potentially more readable way - to generate instances of a base object.

Moving onto the `class` keyword - this is syntactic sugar to resemble OOP in other coding languages.

```js
class CatCreator {
	constructor (name, color){
		this.name = name
		this.color = color
	}
	printIntroduction = function(){
		console.log(`My name is ${this.name}`);
	}
}

const bernice = new CatCreator("Bernice", "gray")
bernice.printIntroduction();
// expected output: "My name is Bernice."
```

This code above does the same exact thing for us (define properties, define functions on the `prototype` object, except is much more readable. However, the work under the hood is the same as the previous example!

# Additional Resources
- [https://www.thecodeship.com/web-development/methods-within-constructor-vs-prototype-in-javascript/](https://www.thecodeship.com/web-development/methods-within-constructor-vs-prototype-in-javascript/)
