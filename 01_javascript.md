## Credits
Kyle Simpson "You Don't Know JS" book series
https://www.amazon.com/Kyle-Simpson/e/B006MAHIQ6/ref=pd_sim_14_bl_1?_encoding=UTF8&refRID=1B2A83TFWJB2QXGGAKJJ

# JavaScript Prototypes and `this`
## All about `this`
* Every function, while its executing, has a reference to a its current execution context, called `this`
* So when we say "context", we usually mean `this`
* How is this context determined? Its based on how and where the function was called.

```js
function someFunc() { /* does stuff */ }

someFunc(); // <--- this is where and how the function was called (call site)
```

### `window` object
* Globally available object
* whenever you define a global variable (one that is not inside a function) it is automatically a property of the `window` object

```js
var bar = 'bar1';
window.bar === bar;
//and sometimes...
this.bar === bar;
```

```js
//when run in a global context, this will be true.
//this MAY be true inside of functions.. but it depends
window === this;
```

### 4 rules for how the `this` keyword is bound
All depend on the **call site**. It's **not** about object oriented or classes.


#### 1. The default binding rule
```js

var bar = "bar1"; // global variable = window.bar = "bar1"

function foo() {  
  console.log(this.bar);
}

function foo2() {
  'use strict';  
  console.log(this.bar);
}

foo() //"bar1"
foo2(); //ERROR: Cannot read property 'bar' of undefined(…)
```

#### 2. The implicit binding rule (aka `something.methodName()`)

In this case, the owning object at the call site (`o2` or `o3`) becomes the `this` keyword

```js
function foo() {  
  console.log(this.bar);
}

var o2 = {
  bar: "bar2",
  foo: foo //this creates a reference to the global function `foo`
};

var o3 = {
  bar: "bar3",
  foo: foo //and this is yet another reference to that same `foo`
};

o2.foo() //"bar2"
o3.foo() //"bar3"
```

#### 3. Explicit binding (at call site)
* using `.call` or `.apply`

```js
var bar = "bar1"; // global variable = window.bar = "bar1"

function foo() {  
  console.log(this.bar);
}
var obj = {bar: "bar2"}

foo() //"bar1"
foo.call(obj) //"bar2"
foo.apply(obj) //"bar2"
```

#### 3.1 Hard binding
* Makes `this` keyword always bound to a specific context no matter where or how its called

```js
var bar = "bar1"; // global variable = window.bar = "bar1"

function foo() {  
  console.log(this.bar);
}

var obj = {bar: "bar2"};

foo = foo.bind(obj); //hard binding

foo(); //"bar2"
foo.call(obj); //"bar2"
foo.apply(obj); //"bar2"
```

#### 4. Using `new`
* When you put `new` in front of a function call, it treats the function call into a constructor call. - ANY function.
* When you put `new` in front, 4 things happen:


1. A brand new object is created
2. Object gets linked to another object
3. That new object is bound as the `this` keyword
4. If that function doesn't otherwise return anything, it will implicitly return the `this` object

```js
function foo() {
  this.baz = "bazzer";
  console.log(this.bar + " " + baz)
}

//foo = foo.bind(window);
var bar = "bar";
var baz = new foo(); //undefined undefined
console.log(baz.baz); //bazzer
```

### Order of Precedence:
1. was it called with the new keyword
2. was it called with call or apply?
3. was there there a containing/owning object
4. default global object

## Prototypes + Object.create()

* https://www.icloud.com/keynote/0_RTDZ3hq8BswO-J-gOeUVMEQ#FED_Training

## Class
* `class` is just syntactic sugar around prototypes. There isn't much magic here.

# Async Code

## Callbacks
* Callbacks and Promises are two ways handling a common problem - asynchronous code.
```js
setTimeout(function A() {
  console.log('hello world')
}, 1000);
```
* Callback hell
```js
setTimeout(function A() {
  setTimeout(function B() {
    setTimeout(function C() {
      console.log('hello world')
    }, 1000);
  }, 1000);
}, 1000);
```
* Can be more than just about nested code. There's ways you can write this code so its not nested
* Its about managing control and being easy to reason about
* Callbacks require you hand control of executing a function to another function.

```js
function myCallback(){
  console.log('Done!')
}

// Inversion of Control: we've handed over control of our program to some 3rd party
// I _trust_ that you (3rd party) will call my callback when I expect, how many times I expect
someThirdPartyLibrary(myCallback);
```
* Node does a lot with Callbacks
```js
fs.readFile('./etc/passwd', function(err, result) {
  if(err){
    console.log(err);
    return;
  }

  //do something with the result
});
```
* Ultimately, if you're using an existing that API (e.g. node stuff), more often then not you just follow their API - be it a callback or promise style. - unless you use something like `promisify`

```js
var promisify = require('promisify-node');
var fs = promisify('fs');

// This function has been identified as an asynchronous function so it has
// been automatically wrapped.
fs.readFile('/etc/passwd').then(function(contents) {
  console.log(contents);
});
```

## Promises
* Promises return a "receipt" or an "IOU"
* Effectively allows us to subscribe to an event to let us know when that function finishes (not unlike listening to a click event on a button)

```js
function getAsyncValue() {
  return new Promise((resolve, reject) => {
    //do any async-code here like fetching data from the server
    setTimeout(() => {
      resolve(42)
    }, 2000)
  })
}

var result = getAsyncValue();
result
  .then(result => console.log('The result is', result));

result
  .then(result => result + 42)
  .then(result => console.log('Muli-then result', result));
//after 2 seconds...
//The result is 42
//Muli-then result 84
```
* Now we own control of when and how are function is executed

## Generators (yield)
* Allows you to pause a function midstream, return, and then resume later on

```js
// Example 1
function* gen(){
  console.log('hello')
  yield null;
  console.log('world')
}

var it = gen();
it.next();
//outputs 'hello'
it.next();
//outputs 'world'

// Example 2
function* gen2(){
  console.log('hello')
  var msg = yield null;
  console.log(msg)
}

var it2 = gen();
it2.next();
//outputs 'hello'
it2.next('dude');
//outputs 'dude'

// Example 3
var it3;
function getData(d) {
  setTimeout(function() {
    console.log(d);
    it3.next(d);
  }, 1000);
}
function* genAsync(){
  yield getData(10);
  yield getData(22);
}

it3 = genAsync();
it3.next();
//pauses 1 second
//10
//pauses 1 second
//22
```
