# Constructor, operator "new"

## My bites

#### Constructor + new => refactor on object creation

Too much repetition
```js
let user1 = {
  name: "Jack",
  isAdmin: false
};

let user2 = {
  name: "John",
  isAdmin: false
};

let user3 = {
  name: "Anne",
  isAdmin: false
};
```
Refactor is good (lazy)
```js run
// any func can be a constructor func
// conventionally, constructor has first char Capitalized to mark itself up
function User(name) { 
  this.name = name;
  this.isAdmin = false;
}

// what really makes a func constructor is new
*!* 
let user1 = new User("Jack");
let user2 = new User("John");
let user3 = new User("Anne");
*/!*

// what would happen without new!!!
let user4 = User("Jack");
console.log(user1);// object
console.log(user4);// undefined
```

#### what `new` does to Constructor func?
```js
// This is constructor func
function User(name) { 
  this.name = name;
  this.isAdmin = false;
}

// Below is what new turned constructor func into 
function User(name) {
*!*
  // this = {};  (implicitly)
*/!*

  // add properties to this
  this.name = name;
  this.isAdmin = false;

*!*
  // return this;  (implicitly)
*/!*
}
```

#### `let user = new function(){...}` is a waste?
```js run
// maybe, one line of code is saved, with this expression
// but such constructor can't be called again in the future
// don't see why we need to do it !!!
let user = new function() {
  this.name = "John";
  this.isAdmin = false;
  // ...other code for user creation
};

console.log(user)
```

#### Make `new` dispensible
```js run
// new.target check whether a func executed with new
function User() {
  console.log(new.target);
}

// without "new":
*!*
User(); // undefined
*/!*

// with "new":
*!*
new User(); // function User { ... }
*/!*
```
```js run
// no more worry about forgetting new!!!
function User(name) {
  if (!new.target) { // if you run me without new
    return new User(name); // ...I will add new for you
  }

  this.name = name;
}

let john = User("John"); // redirects call to new User
alert(john.name); // John
```

#### Interesting effect of `return` in Constructor

```js run
function BigUser() {

  this.name = "John";

  return { name: "Godzilla" };  // <-- returns a specified object
}

console.log( new BigUser().name );  // Godzilla, got that object ^^
```


```js run
function SmallUser() {

  this.name = "John";

  return; // return this (the default object)
  // return (a primitive) has the same effect
}

console.log( new SmallUser().name );  // John
```

#### new Constructor without parentheses

```js run
function User() { // no parameters
  this.name = 'Daniel';
}

let user1 = new User; // <-- no parentheses is fine 
let user2 = new User();
console.log(user1); // content is the same as below
console.log(user2);
console.log(user1 == user2); // different memory place
```

#### add methods to a Constructor function

```js run
function User(name) {
  this.name = name;

  this.sayHi = function() {
    console.log( "My name is: " + this.name );
  };
}

*!*
let john = new User("John");
*/!*

*!*
john.sayHi(); // My name is: John
*/!*

/*
What is inside john then?
john = {
   name: "John",
   sayHi: function() { ... }
}
*/
```


## Original content below

The regular `{...}` syntax allows to create one object. But often we need to create many similar objects, like multiple users or menu items and so on.

That can be done using constructor functions and the `"new"` operator.

## Constructor function

Constructor functions technically are regular functions. There are two conventions though:

1. They are named with capital letter first.
2. They should be executed only with `"new"` operator.

For instance:

```js run
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

*!*
let user = new User("Jack");
*/!*

alert(user.name); // Jack
alert(user.isAdmin); // false
```

When a function is executed as `new User(...)`, it does the following steps:

1. A new empty object is created and assigned to `this`.
2. The function body executes. Usually it modifies `this`, adds new properties to it.
3. The value of `this` is returned.

In other words, `new User(...)` does something like:

```js
function User(name) {
*!*
  // this = {};  (implicitly)
*/!*

  // add properties to this
  this.name = name;
  this.isAdmin = false;

*!*
  // return this;  (implicitly)
*/!*
}
```

So the result of `new User("Jack")` is the same object as:

```js
let user = {
  name: "Jack",
  isAdmin: false
};
```

Now if we want to create other users, we can call `new User("Ann")`, `new User("Alice")` and so on. Much shorter than using literals every time, and also easy to read.

That's the main purpose of constructors -- to implement reusable object creation code.

Let's note once again -- technically, any function can be used as a constructor. That is: any function can be run with `new`, and it will execute the algorithm above. The "capital letter first" is a common agreement, to make it clear that a function is to be run with `new`.

````smart header="new function() { ... }"
If we have many lines of code all about creation of a single complex object, we can wrap them in constructor function, like this:

```js
let user = new function() {
  this.name = "John";
  this.isAdmin = false;

  // ...other code for user creation
  // maybe complex logic and statements
  // local variables etc
};
```

The constructor can't be called again, because it is not saved anywhere, just created and called. So this trick aims to encapsulate the code that constructs the single object, without future reuse.
````

## Constructor mode test: new.target

```smart header="Advanced stuff"
The syntax from this section is rarely used, skip it unless you want to know everything.
```

Inside a function, we can check whether it was called with `new` or without it, using a special `new.target` property.

It is empty for regular calls and equals the function if called with `new`:

```js run
function User() {
  alert(new.target);
}

// without "new":
*!*
User(); // undefined
*/!*

// with "new":
*!*
new User(); // function User { ... }
*/!*
```

That can be used inside the function to know whether it was called with `new`, "in constructor mode", or without it, "in regular mode".

We can also make both `new` and regular calls to do the same, like this:

```js run
function User(name) {
  if (!new.target) { // if you run me without new
    return new User(name); // ...I will add new for you
  }

  this.name = name;
}

let john = User("John"); // redirects call to new User
alert(john.name); // John
```

This approach is sometimes used in libraries to make the syntax more flexible. So that people may call the function with or without `new`, and it still works.

Probably not a good thing to use everywhere though, because omitting `new` makes it a bit less obvious what's going on. With `new` we all know that the new object is being created.

## Return from constructors

Usually, constructors do not have a `return` statement. Their task is to write all necessary stuff into `this`, and it automatically becomes the result.

But if there is a `return` statement, then the rule is simple:

- If `return` is called with object, then it is returned instead of `this`.
- If `return` is called with a primitive, it's ignored.

In other words, `return` with an object returns that object, in all other cases `this` is returned.

For instance, here `return` overrides `this` by returning an object:

```js run
function BigUser() {

  this.name = "John";

  return { name: "Godzilla" };  // <-- returns an object
}

alert( new BigUser().name );  // Godzilla, got that object ^^
```

And here's an example with an empty `return` (or we could place a primitive after it, doesn't matter):

```js run
function SmallUser() {

  this.name = "John";

  return; // finishes the execution, returns this

  // ...

}

alert( new SmallUser().name );  // John
```

Usually constructors don't have a `return` statement. Here we mention the special behavior with returning objects mainly for the sake of completeness.

````smart header="Omitting parentheses"
By the way, we can omit parentheses after `new`, if it has no arguments:

```js
let user = new User; // <-- no parentheses
// same as
let user = new User();
```

Omitting parentheses here is not considered a "good style", but the syntax is permitted by specification.
````

## Methods in constructor

Using constructor functions to create objects gives a great deal of flexibility. The constructor function may have parameters that define how to construct the object, and what to put in it.

Of course, we can add to `this` not only properties, but methods as well.

For instance, `new User(name)` below creates an object with the given `name` and the method `sayHi`:

```js run
function User(name) {
  this.name = name;

  this.sayHi = function() {
    alert( "My name is: " + this.name );
  };
}

*!*
let john = new User("John");

john.sayHi(); // My name is: John
*/!*

/*
john = {
   name: "John",
   sayHi: function() { ... }
}
*/
```

## Summary

- Constructor functions or, briefly, constructors, are regular functions, but there's a common agreement to name them with capital letter first.
- Constructor functions should only be called using `new`. Such a call implies a creation of empty `this` at the start and returning the populated one at the end.

We can use constructor functions to make multiple similar objects.

JavaScript provides constructor functions for many built-in language objects: like `Date` for dates, `Set` for sets and others that we plan to study.

```smart header="Objects, we'll be back!"
In this chapter we only cover the basics about objects and constructors. They are essential for learning more about data types and functions in the next chapters.

After we learn that, we return to objects and cover them in-depth in the chapters <info:prototypes> and <info:classes>.
```
