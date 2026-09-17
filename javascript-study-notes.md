# JavaScript — Complete Study Notes
*Entry-to-Mid-Level Frontend Preparation*

---

## 1. Scope, Hoisting & Closures

### 1.1 Scope — where a variable is visible

Every function creates its own scope. Code can see variables in its own scope and every scope *outside* it (walking outward) — never inward.

```js
let global = "I'm everywhere";

function outer() {
  let outerVar = "outer only";
  function inner() {
    console.log(global);   // ✅ reaches outward to global
    console.log(outerVar); // ✅ reaches outward to outer
  }
  inner();
}
```

This outward-lookup chain is called the **scope chain**.

| | Scope | Hoisting behavior |
|---|---|---|
| `var` | Function-scoped (ignores `{}` blocks) | Hoisted, initialized as `undefined` |
| `let` | Block-scoped | Hoisted, but locked in the Temporal Dead Zone (TDZ) until its line runs |
| `const` | Block-scoped | Same as `let`, plus cannot be reassigned |

### 1.2 Hoisting

Declarations are moved to the top of their scope before code runs.

```js
console.log(a); // undefined — var hoisted, not yet assigned
var a = 5;

console.log(b); // ReferenceError — TDZ blocks access
let b = 10;
```

**Function declarations** are hoisted fully (name + body) — safe to call before they appear:
```js
sayHi(); // ✅ "Hi!"
function sayHi() { console.log("Hi!"); }
```

**Function expressions** are not — only the variable is hoisted, not the function:
```js
sayBye(); // ❌ TypeError: sayBye is not a function
var sayBye = function() { console.log("Bye!"); };
```

> **Remember this:** `var` function expression called early → `TypeError` (exists as `undefined`, can't call `undefined()`).
> `let`/`const` function expression called early → `ReferenceError` (blocked by TDZ, not even accessible).

### 1.3 Closures

A closure happens when an inner function is created inside an outer function and keeps a **live reference** to the outer function's variables — even after the outer function has finished running. Think of it as the inner function carrying a "backpack" of the variables it needs.

```js
function makeCounter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}
const counter = makeCounter(); // makeCounter() has technically finished
counter(); // 1 — backpack still holds count
counter(); // 2 — same backpack, updated in place
```

This is how you get private state without classes — used constantly in debouncing, memoization, and the module pattern.

**Practical example — private bank balance:**
```js
function makeBankAccount(initialBalance) {
  let balance = initialBalance; // private — not reachable except through methods below
  return {
    deposit(amount) { balance += amount; return balance; },
    withdraw(amount) { balance -= amount; return balance; },
    getBalance() { return balance; }
  };
}
```

**Practical example — `once(fn)`, a function that only ever runs once:**
```js
function once(fn) {
  let hasRun = false;
  let result;
  return function (...args) {
    if (!hasRun) {
      result = fn(...args);
      hasRun = true;
    }
    return result;
  };
}
```

### 1.4 The classic `var`-in-loop bug

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// logs: 3 3 3
```
**Why:** `var` is function-scoped, so there is only **one** `i` shared by the whole loop. By the time the callbacks run (100ms later), the loop has finished and `i` is `3`. All three callbacks read the same final value.

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// logs: 0 1 2
```
**Why it works:** `let` gives each loop iteration its own fresh copy of `i`, so each callback closes over a different variable.

**Pre-`let` fix — the IIFE (Immediately Invoked Function Expression):**
```js
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// logs: 0 1 2
```
An IIFE is a function defined and called immediately: `(function(){...})()`. Calling it immediately with `(i)` captures `i`'s *current value* as a new local variable `j`, instead of waiting and reading the shared `var i` later.

### 1.5 Variable shadowing

```js
function outer() {
  var x = 10;
  function inner() {
    console.log(x); // undefined, NOT 10
    var x = 20;
  }
  inner();
}
outer();
```
**Why:** `inner()` has its own local `x` (hoisted to `undefined` at the top of `inner`'s scope), which **shadows** the outer `x = 10`. The scope chain lookup finds the local (not-yet-assigned) `x` first and stops there. This combination — a local hoisted variable hiding an outer one — is called **shadowing**.

> **Common confusion:** this is not "just hoisting" — it's hoisting *combined with* shadowing. If `inner` had no local `x` at all, `console.log(x)` would correctly print `10` from the outer scope.

### 1.6 Closures — Practice

**Beginner**
```js
function greet(greeting) {
  return function(name) {
    return greeting + ", " + name;
  };
}
const sayHello = greet("Hello");
console.log(sayHello("Sam")); // ?
```

**Intermediate**
```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Fix this WITHOUT changing var to let.
```

**Challenging**
Write `once(fn)` (shown above) from scratch without looking.

<details><summary>Solutions</summary>

- Beginner: `"Hello, Sam"`
- Intermediate: wrap the loop body in an IIFE — `(function(j){ setTimeout(() => console.log(j), 100); })(i);`
- Challenging: see 1.3 above.
</details>

---

## 2. `this`, `call`, `apply`, `bind`

### 2.1 The core idea

`this` is not fixed to where a function is *written* — it depends on **how the function is called**. Simplest mental model: `this` = "whoever is directly before the dot when the function is called."

```js
const car = { brand: "Toyota", showBrand() { console.log(this.brand); } };
car.showBrand(); // "Toyota" — called as car.showBrand(), so this = car
```

### 2.2 Losing `this` — the classic bug

```js
const car = { brand: "Toyota", showBrand() { console.log(this.brand); } };
const grabbed = car.showBrand; // just copying the function out
grabbed(); // this is now lost
```
When called with nothing before the dot, `this` falls back to:
- **Strict mode** (ES modules, classes, `"use strict"`): `this` is `undefined` → accessing `this.brand` throws `TypeError: Cannot read properties of undefined`.
- **Non-strict / plain script or Node REPL**: `this` falls back to the global object (`window` in browsers, `global` in Node) → `this.brand` is simply `undefined`, no crash.

> **Remember this:** both behaviors teach the same lesson — copying a method out and calling it "plain" breaks its connection to the original object. Only the *symptom* (crash vs. `undefined`) differs by strictness mode.

### 2.3 Fixing it: `bind`, `call`, `apply`

| Method | Runs immediately? | How args are passed | Returns |
|---|---|---|---|
| `bind(obj)` | No — returns a new function for later | n/a at bind time | A new function with `this` permanently glued to `obj` |
| `call(obj, a, b)` | Yes, immediately | One by one (comma-separated) | The function's return value |
| `apply(obj, [a, b])` | Yes, immediately | As a single array | The function's return value |

```js
function greet(greeting) { console.log(greeting + ", " + this.name); }
const user = { name: "Sam" };

greet.call(user, "Hi");    // "Hi, Sam" — args one by one
greet.apply(user, ["Hi"]); // "Hi, Sam" — args as array
const boundGreet = greet.bind(user);
boundGreet("Hi");          // "Hi, Sam" — this permanently locked
```

**Mnemonic:** `call` → **C**omma-separated args. `apply` → **A**rray of args.

### 2.4 Arrow functions and `this`

Arrow functions have **no `this` of their own** — they inherit `this` from the scope they were *defined* in, not how they're called.

```js
const user = { name: "Sam", greet: () => console.log(this.name) };
user.greet(); // undefined — arrow grabbed 'this' from the outer (module/global) scope, not user
```

This is why arrow functions are the standard fix for the "`this` lost inside a callback" bug:

```js
const user = {
  name: "Zara",
  greetLater() {
    setTimeout(function() {
      console.log("Hi, " + this.name); // regular function — called plainly by setTimeout, this is lost
    }, 100);
  }
};
user.greetLater(); // "Hi, undefined"
```
```js
const user = {
  name: "Zara",
  greetLater() {
    setTimeout(() => {
      console.log("Hi, " + this.name); // arrow — inherits this from greetLater's scope (= user)
    }, 100);
  }
};
user.greetLater(); // "Hi, Zara" ✅
```

> **Important:** this "lost `this`" bug applies to **any** callback passed as a regular function — `setTimeout`, `forEach`, event listeners, etc. — not just `setTimeout` specifically.

### 2.5 `this`/binding — Practice

**Beginner**
```js
const dog = { name: "Rex", bark() { console.log(this.name + " says woof"); } };
const loose = dog.bark;
const fixed = dog.bark.bind(dog);
loose();  // ?
fixed();  // ?
```

**Intermediate**
```js
const player = { name: "Priya" };
function score(points) { console.log(this.name + " scored " + points); }
// Use apply (not call) to print "Priya scored 90"
```

**Challenging**
```js
const timer = {
  seconds: 0,
  start() {
    setInterval(() => {
      this.seconds++;
      console.log(this.seconds);
    }, 1000);
  }
};
timer.start();
// Explain why the arrow function here correctly tracks timer.seconds.
```

<details><summary>Solutions</summary>

- Beginner: `loose()` → `this` falls back to global object (non-strict) or crashes (strict); `fixed()` → `"Rex says woof"`.
- Intermediate: `score.apply(player, [90]);`
- Challenging: the arrow function has no own `this`, so it inherits `this` from `start()`'s scope, where `this` = `timer` (because `start` was called as `timer.start()`). A regular function here would lose that connection.
</details>

---

## 3. Prototypes & Classes

### 3.1 The problem: sharing behavior

Instead of copying the same method onto every object, JS lets objects **share** methods through a **prototype** — think of it as a shared instruction manual all instances check when they need a behavior.

```js
class Dog {
  constructor(name) {
    this.name = name; // each dog's own personal data
  }
  bark() {
    console.log(this.name + " says woof"); // shared, not copied per-instance
  }
}
const dog1 = new Dog("Rex");
const dog2 = new Dog("Milo");
dog1.bark(); // "Rex says woof"
dog2.bark(); // "Milo says woof" — same bark function, different this
```
- `class Dog {...}` — a blueprint.
- `constructor(name)` — runs automatically on `new Dog(...)`, sets up instance-specific data.
- `new Dog("Rex")` — builds an instance from the blueprint.

### 3.2 Inheritance with `extends`

```js
class Dog {
  constructor(name) { this.name = name; }
  bark() { console.log(this.name + " says woof"); }
}
class Puppy extends Dog {
  play() { console.log(this.name + " is playing"); }
}
const puppy = new Puppy("Bruno");
puppy.bark(); // "Bruno says woof" — borrowed from Dog
puppy.play(); // "Bruno is playing" — Puppy's own
```
`extends Dog` means "`Puppy` gets everything `Dog` has, for free." If `Puppy` doesn't have something itself, JS reaches up the **prototype chain** to `Dog` to find it.

### 3.3 Overriding

If the child defines its own version of a method, **the child's version wins** — JS only reaches up to the parent when the child doesn't have its own version.

```js
class Animal { speak() { console.log("Some generic sound"); } }
class Cat extends Animal { speak() { console.log("Meow"); } }
new Cat().speak(); // "Meow"
```

### 3.4 Classes — Practice

**Beginner**
```js
class Shape { area() { console.log("Area not defined"); } }
class Circle extends Shape { area() { console.log("Area = πr²"); } }
new Circle().area(); // ?
```

**Intermediate**
Write a `Vehicle` class with `constructor(brand)` and a `honk()` method logging `"<brand> goes beep"`. Then write `Car extends Vehicle` with an extra `drive()` method.

<details><summary>Solutions</summary>

- Beginner: `"Area = πr²"` — Circle's own method overrides Shape's.
- Intermediate:
```js
class Vehicle {
  constructor(brand) { this.brand = brand; }
  honk() { console.log(this.brand + " goes beep"); }
}
class Car extends Vehicle {
  drive() { console.log(this.brand + " is driving"); }
}
```
</details>

---

## 4. Array Methods — `map`, `filter`, `reduce`

These three cover the vast majority of everyday array work in frontend code — far more common than manual `for` loops.

| Method | Purpose | Returns |
|---|---|---|
| `map` | Transform every item | New array, **same length** |
| `filter` | Keep only items passing a test | New array, **same or shorter** |
| `reduce` | Boil the whole array down to one value | A single value (number, string, object...) |

### 4.1 `map`
```js
const prices = [100, 200, 300];
const withTax = prices.map(price => price * 1.1);
// [110, 220, 330] — original array untouched
```

### 4.2 `filter`
```js
const ages = [12, 25, 17, 40, 8];
const adults = ages.filter(age => age >= 18);
// [25, 40]
```

### 4.3 `reduce`
```js
const nums = [1, 2, 3, 4];
const total = nums.reduce((acc, n) => acc + n, 0);
// 10
```
`acc` (accumulator) is a running value carried forward through each step. The `0` is the starting value.

| step | acc in | n | acc out |
|---|---|---|---|
| 1 | 0 | 1 | 1 |
| 2 | 1 | 2 | 3 |
| 3 | 3 | 3 | 6 |
| 4 | 6 | 4 | 10 |

### 4.4 Chaining `filter` + `map`
```js
const users = [{ name: "Aisha", age: 22 }, { name: "Ravi", age: 17 }, { name: "Meera", age: 30 }];
const eligibleNames = users.filter(u => u.age >= 18).map(u => u.name);
// ["Aisha", "Meera"]
```

> **Typical mistake:** using `> 18` when the requirement says "18 or older" — that silently excludes anyone exactly 18. Use `>= 18` when the wording says "or older."

### 4.5 Array methods — Practice

Given:
```js
const users = [
  { name: "Aisha", age: 22 },
  { name: "Ravi", age: 17 },
  { name: "Meera", age: 30 }
];
```
1. Get an array of just the names (`map`).
2. Get only users 18 or older (`filter`, using `>=`).
3. Get the total combined age (`reduce`).
4. Get the names of only users 18+ (`filter` then `map` chained).

<details><summary>Solutions</summary>

```js
users.map(u => u.name);                              // 1
users.filter(u => u.age >= 18);                       // 2
users.reduce((acc, u) => acc + u.age, 0);              // 3
users.filter(u => u.age >= 18).map(u => u.name);       // 4
```
</details>

---

## 5. Asynchronous JavaScript

### 5.1 The problem

JS normally runs one line at a time, each waiting for the previous one. Some operations (network requests, timers) take real time — if JS froze waiting on them, the whole page would lock up. So JS hands slow work off, keeps running everything else, and comes back later.

```js
console.log("1");
setTimeout(() => console.log("2"), 1000);
console.log("3");
// logs: 1 3 2
```

### 5.2 Promises

A **Promise** is an object representing "something that will finish later, ending in either success or failure." Think of it like a delivery tracking ID.

```js
const orderFood = new Promise((resolve, reject) => {
  const success = true;
  if (success) resolve("Food delivered!");
  else reject("Delivery failed");
});

orderFood
  .then(result => console.log(result))  // runs on resolve
  .catch(error => console.log(error));  // runs on reject
```

### 5.3 `async`/`await`

Syntactic sugar over Promises — makes async code read top-to-bottom instead of chained `.then()`s. Doesn't change what's happening underneath.

```js
function getUser() {
  return new Promise(resolve => resolve({ name: "Aisha" }));
}
async function showUser() {
  const user = await getUser(); // pause here until the Promise settles
  console.log(user.name);
}
```
- `async` on a function → it can use `await` inside, and always returns a Promise itself.
- `await` → pause execution at this line until the Promise resolves/rejects, then continue with the result.

### 5.4 Error handling with `try`/`catch`

```js
function getUser() {
  return new Promise((resolve, reject) => reject("User not found"));
}
async function showUser() {
  try {
    const user = await getUser();
    console.log(user.name);
  } catch (error) {
    console.log("Error:", error); // jumps here on rejection
  }
}
```
An `async` function that `throw`s also produces a rejected Promise, catchable the same way:
```js
async function getData() { throw new Error("fail"); }
getData().catch(err => console.log("caught:", err.message)); // "caught: fail"
```

### 5.5 `fetch` — the real-world pattern

```js
async function loadUsers() {
  try {
    const response = await fetch("https://api.example.com/users"); // await #1: get the response
    const data = await response.json();                            // await #2: parse the body
    console.log(data);
  } catch (error) {
    console.log("Failed to load:", error);
  }
}
```
`fetch` returns a Promise; `.json()` is *also* async (parsing takes time), so both must be awaited.

### 5.6 The event loop — microtasks vs. macrotasks

```js
console.log("1");
setTimeout(() => console.log("2"), 0); // macrotask queue
Promise.resolve().then(() => console.log("3")); // microtask queue
console.log("4");
// logs: 1 4 3 2
```
**Order of operations:**
1. All synchronous code runs first (`1`, `4`).
2. Then the entire **microtask queue** is drained (Promise `.then`/`.catch`/`.finally`, `async`/`await` continuations) — `3`.
3. Only then does JS check the **macrotask queue** (`setTimeout`, `setInterval`) — `2`.

> **Remember this:** even `setTimeout(fn, 0)` never runs synchronously or before microtasks — "0ms" means "as soon as possible after everything currently running (and all microtasks) finish," not "right now."

### 5.7 Async JS — Practice

**Beginner**
```js
function checkAge(age) {
  return new Promise((resolve, reject) => {
    if (age >= 18) resolve("Access granted");
    else reject("Access denied");
  });
}
async function check() {
  try {
    console.log(await checkAge(15));
  } catch (e) { console.log(e); }
}
check(); // ?
```

**Intermediate**
```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
// predict the order
```

<details><summary>Solutions</summary>

- Beginner: `"Access denied"`.
- Intermediate: `A, D, C, B` — sync first, then microtasks, then macrotasks.
</details>

---

## 6. DOM & Events

### 6.1 Selecting and changing elements

```js
const button = document.getElementById("myBtn");
button.textContent = "Clicked!";
button.style.color = "red";
```

### 6.2 Listening for events

```js
button.addEventListener("click", () => {
  console.log("Button was clicked!");
});
```
`addEventListener(type, handler)` wires up a function to run **only when** that event actually happens — nothing runs immediately.

Common event types: `"click"`, `"input"`, `"submit"`, `"mouseover"`.

### 6.3 The `event` object and event delegation

```js
list.addEventListener("click", (event) => {
  console.log("You clicked:", event.target.textContent);
});
```
`event.target` is the exact element that triggered the event — critical when one listener needs to handle clicks on many children.

**Event delegation:** attach **one** listener to a parent instead of one per child. Clicks on children "bubble up" to the parent, and `event.target` tells you which child was actually clicked. Works even for children added dynamically later.

```html
<ul id="fruits"><li>Mango</li><li>Grapes</li><li>Papaya</li></ul>
```
```js
document.getElementById("fruits").addEventListener("click", (event) => {
  console.log(event.target.textContent);
});
```

### 6.4 DOM & Events — Practice

Given:
```html
<button id="likeBtn">Like</button>
<p id="count">0 likes</p>
```
Write JS so clicking `likeBtn` increments a counter and updates `#count`'s text to `"<n> likes"`.

<details><summary>Solution</summary>

```js
const button = document.getElementById("likeBtn");
const likes = document.getElementById("count");
let count = 0;
button.addEventListener("click", () => {
  count++;
  likes.textContent = count + " likes";
});
```
(This relies on the closure concept from Section 1 — `count` persists across clicks because the handler function keeps a reference to it.)
</details>

---

## 7. Debounce & Throttle

### 7.1 The problem

A search box firing a server call on every keystroke wastes requests. A scroll handler firing dozens of times per second is excessive. Both need **rate limiting**, but with different rules.

### 7.2 Debounce — "wait for silence, then run once"

Like an elevator door that resets its closing timer every time someone new walks in — it only closes once nobody's entered for a while.

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);          // cancel the previous pending call
    timer = setTimeout(() => {
      fn(...args);                // only runs if nothing cancels it first
    }, delay);
  };
}
```

**How it works, traced with real calls** (`delay = 500`):
- `debouncedSearch("c")` at 0ms → starts a timer for `search("c")` at 500ms.
- `debouncedSearch("ca")` at 100ms → cancels the "c" timer, starts a new one for `search("ca")` at 600ms.
- `debouncedSearch("cat")` at 150ms → cancels the "ca" timer, starts a new one for `search("cat")` at 650ms.
- No more calls → this last timer survives → `search("cat")` finally runs once, at 650ms.

Only the *last* call that has a full, uninterrupted `delay` after it ever actually fires.

### 7.3 Throttle — "run immediately, then cool down"

Like a camera that can only take one photo per second no matter how fast you mash the shutter.

```js
function throttle(fn, delay) {
  let onCooldown = false;
  return function (...args) {
    if (!onCooldown) {
      fn(...args);
      onCooldown = true;
      setTimeout(() => { onCooldown = false; }, delay);
    }
  };
}
```
- First call: `onCooldown` is `false` → runs immediately, flips to `true`, starts a cooldown timer.
- Any call during cooldown is skipped entirely.
- Once the cooldown timer fires, `onCooldown` resets — the next call runs immediately again.

> **Typical mistake:** placing `onCooldown = true` and the `setTimeout` **outside** the `if` block. That makes every blocked call also reset the cooldown timer, creating overlapping timers instead of one clean cooldown window. Both lines must live **inside** the `if`, since they should only run when `fn` actually fires.

### 7.4 Debounce vs. Throttle

| | Debounce | Throttle |
|---|---|---|
| Rule | Wait for a pause, then run once | Run immediately, then block until cooldown ends |
| Best for | Search boxes, form validation-on-typing | Scroll tracking, resize handlers, button-spam prevention |
| Fires during continuous activity? | No — only after it stops | Yes — at a steady, capped rate |

### 7.5 Understanding `...args` (rest & spread)

`...` **collecting** arguments into an array is called **rest**:
```js
function showAll(...args) { console.log(args); }
showAll(1, 2, 3); // [1, 2, 3]
```
`...` **unpacking** an array back into individual arguments is called **spread**:
```js
function add(a, b, c) { return a + b + c; }
const nums = [1, 2, 3];
add(...nums); // same as add(1, 2, 3) → 6
```
In `debounce`/`throttle`, `...args` collects whatever the wrapper is called with, and `fn(...args)` spreads it back out so the original function receives it exactly as if called directly.

### 7.6 Debounce & Throttle — Practice

**Beginner**
If a user types "cat" quickly (all 3 keystrokes within the debounce delay), how many times does the debounced function run, and with what value?

**Intermediate**
Write `debounce(fn, delay)` from scratch.

**Challenging**
Write `throttle(fn, delay)` from scratch, being careful where `onCooldown = true` and the `setTimeout` are placed.

<details><summary>Solutions</summary>

- Beginner: once, with `"cat"` (the final value).
- Intermediate/Challenging: see 7.2 and 7.3 above.
</details>

---

## 8. ES6+ Essentials

### 8.1 Destructuring

**Objects** — matched by property name:
```js
const user = { name: "Sam", age: 25 };
const { name, age } = user;
```
With a default value (used only if the property is missing/`undefined`):
```js
const { name, age = 18 } = { name: "Sam" }; // age = 18
```
**Arrays** — matched by position:
```js
const colors = ["red", "green", "blue"];
const [first, second] = colors; // "red", "green" — third is simply ignored if not destructured
```

### 8.2 Optional chaining (`?.`)

Stops safely at `null`/`undefined` instead of crashing:
```js
const user = { name: "Sam" };
user.address.city;   // 💥 TypeError
user.address?.city;  // ✅ undefined, no crash
```

### 8.3 Nullish coalescing (`??`)

Falls back **only** when the left side is `null` or `undefined` — unlike `||`, which wrongly treats `0`, `""`, and `false` as "missing" too.

```js
const count = 0;
count || 10;  // 10 — WRONG, 0 is a valid value but || treats it as falsy
count ?? 10;  // 0  — CORRECT, ?? only cares about null/undefined
```

Often combined with optional chaining:
```js
const data = { user: { profile: null } };
data.user.profile?.email ?? "no email"; // "no email"
```

### 8.4 Modules — `import`/`export`

**Named exports** — a file can export multiple named things; import must match exact names and use `{ }`:
```js
// math.js
export function add(a, b) { return a + b; }
```
```js
// app.js
import { add } from "./math.js";
add(2, 3); // 5
```

**Default export** — one main thing per file; no `{ }` on import, and you can name it whatever you like:
```js
// logger.js
export default function log(msg) { console.log(msg); }
```
```js
// app.js
import log from "./logger.js"; // any name works here, but must be used consistently afterward
log("hello");
```

| | Named export | Default export |
|---|---|---|
| Syntax | `export function x() {}` | `export default function x() {}` |
| Import syntax | `import { x } from "..."` | `import anyName from "..."` |
| How many per file | Many | One |
| Name on import | Must match exactly | Your choice |

### 8.5 ES6+ — Practice

```js
const book = { title: "1984" };
// destructure title and author (default "Unknown") in one line
```
```js
const response = { data: { user: { name: "Kiran" } } };
// safely log response.data.user.name and response.data.settings.theme
```
```js
function getQuantity(qty) { return qty || 1; } // fix this bug for qty = 0
```

<details><summary>Solutions</summary>

```js
const { title, author = "Unknown" } = book;
console.log(response.data?.user?.name);       // "Kiran"
console.log(response.data?.settings?.theme);  // undefined
function getQuantity(qty) { return qty ?? 1; }
```
</details>

---

## 9. Error Handling

### 9.1 `try`/`catch`

```js
try {
  JSON.parse("not valid json"); // throws
} catch (error) {
  console.log("Something went wrong:", error.message);
}
// program continues instead of crashing
```

### 9.2 `throw` — triggering your own errors

```js
function withdraw(balance, amount) {
  if (amount > balance) {
    throw new Error("Insufficient funds"); // immediately exits, hands off to nearest catch
  }
  return balance - amount;
}
```

> **Typical mistake:** writing code *after* a `throw` inside the same block, expecting it to run:
```js
if (b === 0) {
  throw new Error("Cannot divide by zero");
  return a / b; // ❌ dead code — never runs, throw already exited the function
}
```

### 9.3 `finally` — cleanup that always runs

```js
try {
  console.log("loading...");
  throw new Error("network failed");
} catch (error) {
  console.log("error:", error.message);
} finally {
  console.log("done loading, hiding spinner"); // runs no matter what — success, failure, or nothing to catch
}
```

### 9.4 Error handling — Practice

```js
function divide(a, b) {
  // throw "Cannot divide by zero" if b is 0, otherwise return a / b
}
// Call divide(10, 0) inside a try/catch/finally and log the error message,
// then log "Operation complete" in finally regardless of outcome.
```

<details><summary>Solution</summary>

```js
function divide(a, b) {
  try {
    if (b === 0) throw new Error("Cannot divide by zero");
    return a / b;
  } catch (error) {
    console.log(error.message);
  } finally {
    console.log("Operation complete");
  }
}
divide(10, 0);
```
</details>

---

## 10. JavaScript Gotchas & Edge Cases

### 10.1 `==` vs `===` — coercion surprises

```js
0 == "0";          // true — "0" → 0
0 == "";           // true — "" → 0
0 == false;        // true — false → 0
"" == false;       // true — both → 0
null == undefined; // true — special mutual case
null == 0;         // false — null does NOT convert to 0
NaN == NaN;         // false — NaN never equals anything, even itself
[] == false;        // true — [] → "" → 0, false → 0, so 0 == 0
```
To check for `NaN`, use `Number.isNaN(value)`, never `==`.

### 10.2 `typeof` quirks

```js
typeof null;         // "object" — a 25+ year old bug, permanent for compatibility
typeof undefined;    // "undefined"
typeof NaN;           // "number" — NaN is a special "not-a-number" NUMBER
typeof [];            // "object" — arrays are just objects to typeof
typeof function(){};  // "function"
```
`typeof` can't distinguish `null` from real objects, or arrays from plain objects. Use `value === null` to check for null specifically, and `Array.isArray(value)` to check for arrays specifically.

### 10.3 Floating-point imprecision

```js
0.1 + 0.2;            // 0.30000000000000004
0.1 + 0.2 === 0.3;     // false
```
Standard binary floating-point math (true across virtually all languages) can't represent most decimals exactly. Never compare decimals with `===`; instead:
```js
function roughlyEqual(a, b) { return Math.abs(a - b) < 0.0001; }
```

### 10.4 Reference vs. value (mutation surprises)

```js
const obj1 = { name: "Sam" };
const obj2 = obj1; // NOT a copy — same object, two labels
obj2.name = "Priya";
obj1.name; // "Priya" — obj1 changed too
```
Objects/arrays are stored **by reference**; primitives (numbers, strings, booleans) are copied **by value**.

**Shallow copy fix** (one level deep only):
```js
const obj2 = { ...obj1 }; // now a genuinely separate top-level object
```

### 10.5 Shallow copy vs. deep copy

Spread only copies the **top level**. Nested objects/arrays inside are still shared references.

```js
const original = { name: "Sam", address: { city: "Delhi" } };
const copy = { ...original };
copy.address.city = "Mumbai";
original.address.city; // "Mumbai" — changed anyway! address was shared, not copied
```
**Deep copy fix:**
```js
const copy = structuredClone(original); // copies at every nesting level
copy.address.city = "Mumbai";
original.address.city; // "Delhi" — untouched now
```

| | Spread `{...obj}` | `structuredClone(obj)` |
|---|---|---|
| Depth | Shallow (top level only) | Deep (all levels) |
| Nested objects/arrays | Still shared references | Fully independent copies |

### 10.6 `setTimeout(fn, 0)` doesn't run immediately

See Section 5.6 — even with `0` delay, the callback always waits for the current synchronous code (and the microtask queue) to finish first.

### 10.7 `+` vs `-`/`*`/`/` — coercion asymmetry

```js
1 + 1;        // 2 — both numbers
"1" + 1;      // "11" — + prefers string concatenation if either side is a string
1 + "1";      // "11"
1 + 1 + "1";  // "21" — (1+1)=2 first, then "2"+"1" = "21", left to right
```
`-`, `*`, `/` **always** mean math — they force both sides to numbers even if one is a string:
```js
"10" - "4"; // 6 — both converted to numbers first, then subtracted
"5" + 3 - 1; // Step 1: "5"+3 → "53" (string concat). Step 2: "53"-1 → 52 (forced numeric)
```
Arrays with `+` also fall back to string conversion:
```js
[1,2] + [3,4]; // "1,23,4" — arrays auto-convert to comma-joined strings, then concatenate
```

### 10.8 Full Gotchas Summary Table

| Gotcha | Key rule |
|---|---|
| `==` vs `===` | `==` coerces types before comparing; `===` never does |
| `typeof null` | Returns `"object"` — a permanent historical bug |
| `typeof` arrays | Also `"object"` — use `Array.isArray()` to actually detect arrays |
| Floating point | `0.1 + 0.2 !== 0.3` — compare with a tolerance, not `===` |
| Objects/arrays | Copied by reference, not by value — mutating one mutates all references |
| Shallow vs deep copy | Spread copies one level; `structuredClone()` copies all levels |
| `var` in loops | One shared variable across all iterations — use `let` or an IIFE |
| `setTimeout(fn, 0)` | Still waits for sync code + microtasks to finish first |
| `+` coercion | Prefers string concatenation if either side is a string |
| `-`/`*`/`/` coercion | Always force numeric conversion, even on strings |
| `this` in callbacks | Regular functions lose `this` when called plainly (e.g. by `setTimeout`) — use arrow functions |

### 10.9 Gotchas — Practice

```js
console.log(null == undefined);
console.log(null === undefined);
```
```js
const arr = [1, 2, 3];
const arrCopy = [...arr];
arrCopy.push(4);
console.log(arr, arrCopy);
```
```js
const config = { level: 1, sub: { retries: 3 } };
const copy = { ...config };
copy.sub.retries = 5;
console.log(config.sub.retries);
```
```js
const team = { name: "Falcons", showName() {
  setTimeout(function() { console.log("Team: " + this.name); }, 100);
}};
team.showName();
// predict, explain, then fix so it logs "Team: Falcons"
```

<details><summary>Solutions</summary>

```js
null == undefined;  // true
null === undefined; // false

arr;      // [1, 2, 3]      — untouched
arrCopy;  // [1, 2, 3, 4]   — separate array via spread

config.sub.retries; // 5 — shallow copy, "sub" object was shared

// team.showName() logs "Team: undefined" — regular function loses `this` when called plainly by setTimeout.
// Fix: use an arrow function so `this` is inherited from showName's scope:
const team2 = { name: "Falcons", showName() {
  setTimeout(() => { console.log("Team: " + this.name); }, 100);
}};
```
</details>

---

## 11. My Common Mistakes & Weak Areas

Based on the actual practice through this session:

- **Typos in code that break functionality** — `wihtdraw` instead of `withdraw`, `error.mrssage` instead of `error.message`. These are exactly the kind of bugs that silently break real apps (calling a misspelled method name throws "not a function"; a misspelled property just silently returns `undefined`).
- **Dead code after `throw`** — writing a `return` statement immediately after a `throw` inside the same block, not realizing `throw` already exits the function, so that line can never execute.
- **Testing the wrong case** — writing a function correctly but calling it with input that doesn't actually exercise the bug/edge case being tested (e.g. calling `divide(1, 10)` when the assignment asked to test the divide-by-zero path).
- **`>` vs `>=` in "at least" conditions** — using `age > 18` when the requirement says "18 or older," which silently produces wrong results exactly at the boundary value.
- **`...args` (rest/spread) was initially unfamiliar** — needed a from-scratch breakdown (Section 1 building block → Section 3 rest/spread step) before the debounce/throttle implementations made sense. Once explained in isolation, this was picked up solidly.
- **String concatenation with mixed operators** — occasionally dropped a character while tracing left-to-right `+` chains by hand (e.g. predicting `"13"` instead of the correct `"123"` for `1 + "2" + 3`). The underlying rule (left-to-right, string wins once either side is a string) was understood — the slip was in careful manual tracing, not the concept.
- **`this` inside plain callbacks** required a slower, rebuilt-from-zero explanation (Section 2.2–2.4) before it was fully solid — but once explained step-by-step with the "who's before the dot" framing, all subsequent quiz questions on this topic (including `forEach` callbacks, not just `setTimeout`) were answered correctly and with correct reasoning.
- **Throttle implementation bug** — placed `onCooldown = true` and the `setTimeout(...)` outside the `if (!onCooldown)` block instead of inside it, which would cause every blocked call to also reset/extend the cooldown timer. Correctly understood once the bug was pointed out.

**Areas that were consistently strong:** closures and the `var`-in-loop fix (solved independently before it was formally taught), `map`/`filter`/`reduce` chaining, async/Promise/`try`-`catch` flow, event delegation, debounce/throttle *conceptual* understanding, destructuring/optional chaining/nullish coalescing, and the microtask-vs-macrotask event loop ordering.

---

## 12. Quick-Reference Cheat Sheet

### Variables
```js
var x;    // function-scoped, hoisted as undefined
let x;    // block-scoped, TDZ until declared
const x;  // block-scoped, TDZ, cannot be reassigned
```

### Functions
```js
function name() {}          // declaration — fully hoisted
const name = function() {}; // expression — not hoisted
const name = () => {};      // arrow — no own `this`, not hoisted
```

### `this` binding
```js
fn.call(obj, a, b);   // run now, args individually
fn.apply(obj, [a, b]); // run now, args as array
const bound = fn.bind(obj); // new function, this locked, run later
```

### Array methods
```js
arr.map(x => ...)      // transform each → new array, same length
arr.filter(x => ...)   // keep matching → new array, shorter/equal
arr.reduce((acc, x) => ..., initial) // → single value
```

### Object/array copying
```js
const shallow = { ...obj };        // top level only
const shallowArr = [...arr];
const deep = structuredClone(obj); // all levels
```

### Destructuring & modern operators
```js
const { a, b = default } = obj;   // object destructuring + default
const [x, y] = arr;               // array destructuring
obj.a?.b;                          // optional chaining
value ?? fallback;                 // nullish coalescing (only null/undefined)
function f(...args) {}             // rest — collect into array
f(...arr);                         // spread — unpack array into args
```

### Async
```js
new Promise((resolve, reject) => { ... });
promise.then(onSuccess).catch(onError).finally(onAlways);

async function f() {
  try {
    const result = await somePromise;
  } catch (error) { ... }
}

const response = await fetch(url);
const data = await response.json();
```

### Error handling
```js
try {
  // risky code
} catch (error) {
  // handle it
} finally {
  // always runs
}
throw new Error("message");
```

### DOM & Events
```js
document.getElementById("id");
el.textContent = "...";
el.style.property = "...";
el.addEventListener("click", (event) => { event.target; });
```

### Modules
```js
export function x() {}       // named export
export default function x(){} // default export
import { x } from "./file.js";  // named import — exact name, curly braces
import anyName from "./file.js"; // default import — any name, no braces
```

### Debounce / Throttle skeletons
```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

function throttle(fn, delay) {
  let onCooldown = false;
  return (...args) => {
    if (!onCooldown) {
      fn(...args);
      onCooldown = true;
      setTimeout(() => { onCooldown = false; }, delay);
    }
  };
}
```

---

## 13. Learning Roadmap

### Beginner
- ✅ Variables (`var`/`let`/`const`) and basic scope
- ✅ Functions, function expressions vs. declarations
- ✅ Basic objects and arrays
- ✅ `if`/loops fundamentals *(assumed prior knowledge — not explicitly re-covered in this session, since the quizzes confirmed solid basics)*

### Intermediate
- ✅ Closures (including the classic `var`-in-loop bug and its fixes)
- ✅ Hoisting nuances (declarations vs. expressions, TDZ, shadowing)
- ✅ `this`, `call`/`apply`/`bind`, arrow function `this` inheritance
- ✅ Classes, `extends`, method overriding
- ✅ `map`/`filter`/`reduce` and chaining
- ✅ Destructuring, rest/spread, optional chaining, nullish coalescing
- ✅ `try`/`catch`/`finally`/`throw`
- ✅ Modules (named vs. default import/export)

### Advanced
- ✅ Promises, `async`/`await`, `fetch` pattern
- ✅ Event loop — microtask vs. macrotask ordering
- ✅ DOM events, event delegation, `event.target`
- ✅ Debounce and throttle (from-scratch implementations)
- ✅ `==`/`===` coercion edge cases, `typeof` quirks, floating-point imprecision
- ✅ Shallow vs. deep copy (`structuredClone`)
- ✅ `this` inside plain-function callbacks (the `setTimeout`/`forEach` trap)
- ⚠️ Full event-loop internals beyond microtask/macrotask ordering (e.g. rendering steps, requestAnimationFrame) — touched only at the level needed for the ordering questions
- ❌ React or any frontend framework — explicitly deferred, not part of this session
- ❌ Generators, `Symbol`, `Map`/`Set`, Proxies — not covered in this session
- ❌ TypeScript — not covered in this session

