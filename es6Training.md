# ES6

## Module
import and export
```
// export
export function sum(){
  console.log('sum');
}
export const API = 'api';
export default {a: 'a'}

// import
import obj, {sum, API} from './export';
```

## Scope
let and const

The let statement declares a block scope local variable

Constants are block-scoped, much like variables defined using the let statement. The value of a constant **cannot change through reassignment, and it can't be redeclared.**

```
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

let a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6

(function f1() {
  var n = 5;
  if (1 === 1) {
    var n = 10;
  }
  console.log(n); // 10
})();

(function f1() {
  let n = 5;
  if (1 === 1) {
    let n = 10;
  }
  console.log(n); // 5
})();

```


## Set and Map

The Set object lets you store unique values of any type, whether primitive values or object references.

```
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
```

remove duplicate values from array.

```

function unique(arr) {
  return [...new Set(arr)];
}

```

## Destructuring

The destructuring assignment syntax is a JavaScript expression that makes it possible to unpack values from arrays, or properties from objects, into distinct variables.

```
let [a, b, c] = [1, 2, 3];
console.log(a, b, c);
let [foo, [[bar], baz]] = [1, [[2], 3]];
console.log(foo, bar, baz);

/**
 * @description use destructuring on something iterable
 */
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
console.log(first, second, third, fourth, fifth, sixth);

/**
 * @description default value
 */
let [x, y = 'b'] = ['a'];


let { age, name } = { age: 13, name: "John" };
console.log(age, name);

let { age: years, name: alias } = { age: 13, name: "John" };
console.log(years, alias);

const [l1, l2, l3, l4, l5] = 'hello';
console.log(l1, l2, l3, l4, l5);

/**
 * @description change x, y
 * @type {number}
 */
let m = 1;
let n = 2;

[m, n] = [n, m];
console.log(m, n);
```

## Iterator

Whenever an object needs to be iterated (such as at the beginning of a for..of loop), its @@iterator method is called with no arguments, and the returned iterator is used to obtain the values to be iterated.

Some built-in types have a default iteration behavior, while other types (such as Object) do not.

### Interface

```

interface Iterable {
   [Symbol.iterator]() : Iterator,
}

```

```

interface Iterator {
    next(value?: any) : IterationResult,
}

```

```

interface IterationResult {
    value: any,
    done: boolean,
}

```

The Symbol.iterator well-known symbol specifies the default iterator for an object. Used by for...of.

### Make an object iterable

```
class IterableObject {
  constructor() {
    this.index = 0;
    this.uis = ['Joy', 'Josie', 'Carl', 'Orland', 'Wisdom', 'Lucy'];
  }

  [Symbol.iterator]() {
    return this;
  }

  next() {
    const done = this.index === this.uis.length;
    return {
      value: this.uis[this.index++],
      done
    }
  }
}

let itg = new IterableObject();

for (let i of itg) {
  console.log(i);
}
```

### Execute iterator

```
function iterate(iterator, value) {
  let result = iterator.next(value);
  console.log(result);
  if (result.done) return result.value;
  iterate(iterator, result.value);
}

iterate(new GUI());
```

### Assign a generator to [Symbol.iterator]

```
let fibonacci = {
  [Symbol.iterator]: function*() {
    let index = 0;
    for (; ;) {
      yield index++;
    }
  }
};

for (let n of fibonacci) {
  // truncate the sequence at 1000
  if (n > 10) break;
  console.log(n);
}
```

## Generator

The Generator object is returned by a generator function and it conforms to both the
 [iterable protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterable_protocol)
                    and the
 [iterator protocol.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterator_protocol)

### Syntax

\* and yield

```
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

let ge = function*() {
  let index = 0;
  for (; ;) {
    yield index++;
  }
}();

console.log(ge.next());

for (let i of ge) {
  if (i > 10) break;
  console.log(i);
}


```

### How to prove that a generator is both iterator and iterable

```
console.log(typeof ge.next);
// "function", because it has a next method, so it's an iterator
console.log(typeof ge[Symbol.iterator]);
// "function", because it has an @@iterator method, so it's an iterable
console.log(ge[Symbol.iterator]() === ge);
// true, because its @@iterator method returns itself (an iterator), so it's an well-formed iterable
```

## Proxy

The Proxy object is used to define custom behavior for fundamental operations.

### Syntax

```

var p = new Proxy(target, handler);

```

### Handler

An object whose properties are functions which define the behavior of the proxy when an operation is performed on it.

```
// basic usage
let proxy = new Proxy({}, {
  get(target, property){
    console.log(`get property ${property}`);
    return property;
  }
});

console.log(proxy.a);
console.log(proxy.b);
console.log(proxy.any);

// advance usage
const dom = new Proxy({}, {
  get(target, property) {
    return function (attrs = {}, ...children) {
      const el = document.createElement(property);
      for (let prop of Object.keys(attrs)) {
        el.setAttribute(prop, attrs[prop]);
      }
      for (let child of children) {
        if (typeof child === 'string') {
          child = document.createTextNode(child);
        }
        el.appendChild(child);
      }
      return el;
    }
  }
});

const el = dom.div({},
  'Hello, my name is ',
  dom.a({ href: '//example.com' }, 'Mark'),
  '. I like:',
  dom.ul({},
    dom.li({}, 'The web'),
    dom.li({}, 'Food'),
    dom.li({}, '…actually that\'s it')
  )
);

console.log(el.innerHTML);
```

## Decorator

Decorators make it possible to annotate and modify classes and properties at design time.
A decorator is:

* an expression
* that evaluates to a function
* that takes the target, name, and decorator descriptor as arguments
* and optionally returns a decorator descriptor to install on the target object

Consider a simple class definition:

```

class Person {
  name() { return `${this.first} ${this.last}` }
}

```

Evaluating this class results in installing the name function onto Person.prototype, roughly like this:

```

Object.defineProperty(Person.prototype, 'name', {
  value: specifiedFunction,
  enumerable: false,
  configurable: true,
  writable: true
});

```

A decorator precedes the syntax that defines a property:

```

class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}

```

Now, before installing the descriptor onto Person.prototype, the engine first invokes the decorator:

```

let description = {
  type: 'method',
  initializer: () => specifiedFunction,
  enumerable: false,
  configurable: true,
  writable: true
};

description = readonly(Person.prototype, 'name', description) || description;
defineDecoratedProperty(Person.prototype, 'name', description);

function defineDecoratedProperty(target, property, { initializer, enumerable, configurable, writable }) {
  Object.defineProperty(target, property, { value: initializer(), enumerable, configurable, writable });
}

```

This has an opportunity to intercede before the relevant defineProperty actually occurs.

# Thanks
