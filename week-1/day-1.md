Here's the detailed Q&A for **Day 1: JavaScript Core**, with explanations and code snippets for each.

---

## 1. Closures

**Q: What is a closure? Give an example.**

A closure is when a function "remembers" the variables from its outer scope even after that outer function has finished executing.

```javascript
function outer() {
  let count = 0; // this variable is "enclosed"
  return function inner() {
    count++;
    return count;
  };
}

const counter = outer();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

`inner` keeps access to `count` even though `outer()` already returned. This is the basis of things like private variables, memoization, and the module pattern.

**Follow-up Q: Classic interview trap — what does this print?**

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3, 3, 3
```

Because `var` is function-scoped, all three callbacks share the same `i`, which is `3` by the time the callbacks run.

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2
```

`let` is block-scoped, so each loop iteration gets its own `i` captured in a new closure.

---

## 2. Hoisting

**Q: What is hoisting? How do `var`, `let`, and `const` behave differently?**

JavaScript moves declarations (not initializations) to the top of their scope during compilation.

```javascript
console.log(a); // undefined (declaration hoisted, not the assignment)
var a = 5;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 10;
```

- `var` → hoisted and initialized with `undefined`.
- `let`/`const` → hoisted but stay in the **Temporal Dead Zone (TDZ)** until the actual line executes. Accessing them before that throws an error.

**Follow-up Q: Are function declarations hoisted with their body?**

```javascript
sayHi(); // "Hi!" — works fine

function sayHi() {
  console.log("Hi!");
}
```

Yes — function declarations are fully hoisted (name + body). Function **expressions** are not:

```javascript
sayHello(); // TypeError: sayHello is not a function
var sayHello = function () {
  console.log("Hello");
};
```

---

## 3. `this` Binding (call, apply, bind)

**Q: How does `this` get determined in JavaScript?**

`this` depends on **how a function is called**, not where it's defined.

```javascript
const user = {
  name: "Ravi",
  greet() {
    console.log(`Hi, I'm ${this.name}`);
  },
};

user.greet(); // "Hi, I'm Ravi" — this = user (called on the object)

const fn = user.greet;
fn(); // "Hi, I'm undefined" — this = undefined/global (called standalone)
```

**Q: Difference between `call`, `apply`, and `bind`?**

```javascript
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const person = { name: "Priya" };

greet.call(person, "Hello"); // invokes immediately, args passed individually
greet.apply(person, ["Hi"]); // invokes immediately, args passed as an array
const boundGreet = greet.bind(person); // returns a NEW function, doesn't invoke
boundGreet("Hey"); // "Hey, Priya"
```

**Practical backend use case:** `bind` is common when passing class methods as Express route handlers, so `this` doesn't get lost:

```javascript
class UserController {
  constructor() {
    this.getUser = this.getUser.bind(this); // preserves `this` inside getUser
  }
  getUser(req, res) {
    res.send(this.someProperty);
  }
}
```

---

## 4. Prototypal Inheritance

**Q: How does prototypal inheritance work in JS?**

Every object has an internal link (`[[Prototype]]`) to another object it can inherit properties/methods from.

```javascript
const animal = {
  eat() {
    console.log(`${this.name} is eating`);
  },
};

const dog = Object.create(animal); // dog's prototype = animal
dog.name = "Rex";
dog.eat(); // "Rex is eating" — eat() is found via the prototype chain
```

With ES6 classes (syntactic sugar over the same mechanism):

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  eat() {
    console.log(`${this.name} is eating`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`${this.name} says Woof!`);
  }
}

const rex = new Dog("Rex");
rex.eat(); // inherited from Animal
rex.bark(); // own method
```

---

## 5. `var` vs `let`/`const` and Temporal Dead Zone (TDZ)

**Q: Explain scoping differences and TDZ.**

|                        | `var`                 | `let` / `const`          |
| ---------------------- | --------------------- | ------------------------ |
| Scope                  | Function-scoped       | Block-scoped             |
| Redeclare              | Allowed               | Not allowed (same scope) |
| Hoisting               | Hoisted + `undefined` | Hoisted but in TDZ       |
| Global object property | Yes (`window.x`)      | No                       |

```javascript
{
  console.log(x); // undefined
  var x = 1;
}

{
  console.log(y); // ReferenceError (TDZ)
  let y = 1;
}
```

---

## 6. Deep vs Shallow Copy

**Q: Difference between shallow copy and deep copy? How do you do each?**

**Shallow copy** — copies only the top level; nested objects are still shared by reference.

```javascript
const original = { name: "Amit", address: { city: "Mumbai" } };

const shallow = { ...original }; // or Object.assign({}, original)
shallow.address.city = "Delhi";

console.log(original.address.city); // "Delhi" — changed! (nested object was shared)
```

**Deep copy** — fully independent copy, including nested objects.

```javascript
const deep = structuredClone(original); // modern, built-in (Node 17+)
// or: const deep = JSON.parse(JSON.stringify(original)); // older approach, loses functions/undefined/dates

deep.address.city = "Pune";
console.log(original.address.city); // "Delhi" — unaffected
```

**Note for interviews:** mention `JSON.parse(JSON.stringify())` breaks for `Date`, `undefined`, functions, and `Map`/`Set` — `structuredClone` or a library like `lodash.cloneDeep` is the safer answer.

---

## 7. The Event Loop (this is the big one at 2 YOE)

**Q: Explain the event loop with an example that prints in an unexpected order.**

```javascript
console.log("1: Start");

setTimeout(() => console.log("2: setTimeout"), 0);

Promise.resolve().then(() => console.log("3: Promise"));

console.log("4: End");
```

**Output:**

```
1: Start
4: End
3: Promise
2: setTimeout
```

**Why:**

1. Synchronous code (`console.log`) runs first — it's on the **call stack**.
2. `setTimeout` callback goes into the **macrotask queue** (even with `0ms` delay, it waits its turn).
3. `Promise.then` callback goes into the **microtask queue**.
4. After the call stack is empty, the event loop **drains all microtasks first**, then picks one macrotask.

So order is: all sync code → all microtasks → one macrotask → repeat.

**Q: Difference between `process.nextTick()`, `Promise.resolve().then()`, and `setImmediate()`?**

```javascript
console.log("start");

setImmediate(() => console.log("setImmediate"));

process.nextTick(() => console.log("nextTick"));

Promise.resolve().then(() => console.log("promise"));

console.log("end");
```

**Output:**

```
start
end
nextTick
promise
setImmediate
```

- `process.nextTick()` — Node-specific, runs **before** any other microtask, even before promises. Highest priority.
- `Promise.then()` — standard microtask queue, runs after all `nextTick` callbacks but before macrotasks.
- `setImmediate()` — a macrotask, runs in the "check" phase of the event loop, generally after I/O callbacks.

**Danger to mention:** overusing `process.nextTick()` recursively can starve the event loop (I/O never gets a chance to run) — a real gotcha interviewers like to probe.

---

## 8. `await` Inside `forEach` — the classic Node.js trap

**Q: What happens when you `await` inside a `forEach` loop? Why doesn't it work as expected?**

```javascript
async function processItems(items) {
  items.forEach(async (item) => {
    await saveToDB(item); // this does NOT pause the outer function
  });
  console.log("All done!"); // this runs BEFORE the DB saves complete
}
```

**Why it fails:** `forEach` doesn't know or care that its callback is `async`. It fires all callbacks immediately and moves on — it never awaits them. So `"All done!"` logs before any `saveToDB` calls resolve.

**Correct approaches:**

```javascript
// Option 1: sequential (one after another)
async function processItemsSequential(items) {
  for (const item of items) {
    await saveToDB(item); // properly pauses each iteration
  }
  console.log("All done!");
}

// Option 2: parallel (all at once, faster if independent)
async function processItemsParallel(items) {
  await Promise.all(items.map((item) => saveToDB(item)));
  console.log("All done!");
}
```

This is an extremely common real-world bug in Express route handlers (e.g., looping over an array to insert into MongoDB), so interviewers love asking it.

---

### Quick Self-Test (try answering without scrolling up)

1. Why does `let` in a `for` loop behave differently from `var` when used inside a `setTimeout`?
2. What's the actual execution order of `process.nextTick`, microtasks, and macrotasks?
3. Why does `bind()` return a new function instead of calling it immediately?
4. Give a real backend scenario where a shallow copy could cause a bug.
5. Fix this code so all DB writes complete before logging "done":

```javascript
users.forEach(async (u) => await User.create(u));
console.log("done");
```

Want me to go through **Day 2 (Async JS Deep Dive)** in the same format next?
