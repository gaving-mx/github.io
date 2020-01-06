# Javascript

## Why we learn javascript?
1. Relative easy to learn for someone who does not know anything about programming. 
for someone who knows programming. With javascript it is more important to understand how it works. 
It’s easy to learn but difficult to master.

2. The web would be empty without it. A staggering 94.7% of all sites on the web use JavaScript. 
The remaining 5.3% would probably be static GeoCities homepages built in 1994.

3. Easily available Packages. Don’t reinvent the wheel. You can find a package for anything generic thing you can think of.

5. More possibilities of finding your answer on Stack Overflow.

5. [Because developers say so](https://octoverse.github.com/2017/)
6. [Because developers say it again](https://octoverse.github.com/projects)

## What is javaScript

JavaScript is a programming language that adds interactivity to your website. 
There are also more advanced server side versions of JavaScript such as Node.Js

## How was javascript born
[JavaScript inventor Brendan Eich](https://www.youtube.com/watch?v=IPxQ9kEaF8c)

## What's the future of javaScript

## Learning Javascript
### Data Structure
Boolean, String, Number, Null, Undefined, Object

### Data Type Conversion
![data-type-conversion.png](https://upload-images.jianshu.io/upload_images/2261833-89fd1efc12489f86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Identify Data Type
#### typeof
The typeof operator returns a string indicating the type of the unevaluated operand.
```
console.log(typeof 1); // number
console.log(typeof 'string'); // string
console.log(typeof true); // boolean
console.log(typeof null); // object
console.log(typeof undefined); // undefined
console.log(typeof {}); // object
console.log(typeof NaN); // number
console.log(typeof function () {
}); // function
```

#### instanceof
The instanceof operator tests whether the prototype property of a constructor appears anywhere in the prototype chain of an **object**.
**Primitive value will return false**
```
instanceof actually does
new Number(123).__proto__ === Number.prototype
```

```
console.log('123 instanceof Number: ', 123 instanceof Number); // false
console.log('new Number(123) instanceof Number: ', new Number(123) instanceof Number); // true
console.log('Number(123) instanceof Number: ', Number(123) instanceof Number); // false
```
#### Object.prototype.toString
Every object has a toString() method that is automatically called when the object is to be represented as a string. toString() returns "[object type]", where type is the object type.
```
console.log('[] is Array: ', Object.prototype.toString.call([]) === "[object Array]");
console.log('{} is Object: ', Object.prototype.toString.call({}) === "[object Object]");
console.log('null is not an Object: ', Object.prototype.toString.call(null) === "[object Null]");
console.log('undefined is not an Object: ', Object.prototype.toString.call(undefined) === "[object Undefined]");
console.log('1 is Number: ', Object.prototype.toString.call(1) === "[object Number]");
console.log('true is Boolean: ', Object.prototype.toString.call(true) === "[object Boolean]");
console.log('\'string\' is String: ', Object.prototype.toString.call('string') === "[object String]");
```

### Object
Pairs of names (strings) and values (any value) where the name is separated from the value by a colon.
```
var o = {
    p: "Hello World"
};
```

#### Way to create object
```
var o1 = {};
var o2 = new Object();
var o3 = Object.create(null);
```

#### Read and write property
```
var o = {
    p: "Hello World"
};
o.p // "Hello World"
o['p'] // "Hello World"
```

#### Delete property
```
var o = { p:1 };
delete o.p // true
o.p // undefined
```

### Function
A function is a code snippet that can be called by other code or by itself.

#### Way to create function
```
function print(){
    // ...
}
var print = function (){
    // ...
};
var add = new Function("x","y","return (x+y)");
```

#### Hoisting

#### Closure
```
function f() {
    var v = 1;
    var c = function (){
        return v;
    };
    return c;
}
var o = f();
o(); // 1
```

#### IIFE
```
(function(){ /* code */ }());
//or
(function(){ /* code */ })();
```

#### Recursive
```
function loop(x) {
   if (x >= 10)
      return;
   loop(x + 1);
};
```

### Array
An array is an ordered collection of data, it can hold different types of data.
Each item in an array has a number attached to it, called a numeric index, that allows you to access it. In JavaScript, arrays start at index zero.
```
[1, 'a', {}]
```

#### Way to create array
```
[element0, element1, ..., elementN]
new Array(element0, element1[, ...[, elementN]])
new Array(arrayLength)
```

#### Loop over an Array
* for
* forEach
* map
* reduce

### Prototype
When it comes to inheritance, JavaScript only has one construct: objects. 
Each object has a private property which holds a link to another object called its prototype.

prototype exist on constructor, __proto__ exists on instance

```
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}

var auto = new Car('Honda', 'Accord', 1998);

console.log(Car.prototype);
console.log(auto.__proto__);
console.log(auto.constructor.prototype);
console.log(Object.getPrototypeOf(auto) === Car.prototype);
console.log(Object.getPrototypeOf(auto) === auto.__proto__);
console.log(Object.getPrototypeOf(auto) === auto.constructor.prototype);
console.log(auto instanceof Car);
console.log(auto instanceof Object);
```

```
function C() {
}
function D() {
}
var o = new C();
console.log('o instanceof C: ', o instanceof C);
D.prototype = o;
var q = new D();
console.log('q instanceof D: ', q instanceof D);
console.log('q instanceof C: ', q instanceof C);

C.prototype = {};
console.log('o instanceof C: ', o instanceof C); // false
```

```
function Animal(name) {
  this.name = name || 'animal';
}
Animal.prototype.eat = function eat(food = 'food') {
  console.log(`${this.name} is eating ${food}`)
};

function Cat(name) {
  Animal.call(this, name);
}

// Cat.prototype = new Animal();
Cat.prototype = Object.create(Animal.prototype);
Cat.prototype.constructor = Cat;


var cat = new Cat('cat');
cat.eat('fish');
console.log(cat.constructor);
```

```
function Foo(){}
// So, when you call
var o = new Foo();
// JavaScript actually just does
var o = new Object();
o.[[Prototype]] = Foo.prototype;
Foo.call(o);
```

### Promise
