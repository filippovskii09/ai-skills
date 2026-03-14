# JS Senior Expertise: The Source of Truth

This document serves as the absolute "Source of Truth" for JavaScript and TypeScript development at a **Senior+ / Principal level**. It defines development standards through a dual lens: **How the Machine (V8 Engine) sees the code** versus **How a Human (Expert Engineer) reads the code**. 

You MUST adhere to these low-level constraints and high-level structural rules to ensure code is highly optimized, predictable, and strictly coherent.

---

## 1. V8 Engine & Low-Level Optimization

To write performant JavaScript, you must understand how the V8 engine interprets, optimizes, and potentially deoptimizes your code via JIT compilation (Ignition interpreter and TurboFan compiler).

### 1.1 Hidden Classes (Shapes) and Object Initialization
JavaScript is a dynamically typed language, but V8 uses static structures internally called **Hidden Classes (Shapes)**. V8 attaches a Shape to every object to track property layout, enabling rapid property access.

- **Transition Graph**: Modifying objects dynamically creates branching in the Shape transition graph. Objects that share the same Shape are optimized together.
- **Rule**: Initialize all object properties in the same order, preferably in the constructor. 
- **Deoptimization**: Never use the `delete` keyword on objects. It breaks the Hidden Class chain, forcing V8 to downgrade the object to a slower "dictionary mode" (hash table). Instead, assign `null` or `undefined`.

```javascript
// ✅ GOOD: Monomorphic Shape (properties initialized in exact same order)
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.z = null; // Pre-allocating memory for z
  }
}

// ❌ BAD: Breaks transition chain, creates polymorphic Shapes
const p1 = new Point(1, 2);
p1.z = 3; 
const p2 = new Point(1, 2);
p2.w = 4;
delete p1.x; // Fatal for performance: triggers dictionary mode transition
```

### 1.2 Inline Caches (IC)
When V8 looks up a property on an object, it performs a costly lookup. **Inline Caches** memorize the memory offset of properties matching a specific Shape. Subsequent lookups for objects with the identical Shape bypass the lookup process completely, accessing memory directly.

### 1.3 Function States & Call Sites
TurboFan aggressively optimizes functions based on the types of arguments they receive. Call sites transition between optimization states:
- **Monomorphic (1 Shape)**: Extremely fast. The IC stores a single memory offset. TurboFan can heavily inline and compile to raw machine code.
- **Polymorphic (2-4 Shapes)**: Fast, but requires a switch statement at the machine code level. The IC stores multiple offsets.
- **Megamorphic (5+ Shapes)**: Slowest. V8 bails out of ICs entirely, falling back to global hash table lookups. TurboFan aborts optimization.

```javascript
function processUser(user) { /* ... */ }

// ✅ Monomorphic: `user` always has the exact same Shape.
processUser({ id: 1, name: "A" });
processUser({ id: 2, name: "B" });

// ❌ Megamorphic: V8 will bail out of optimization after 5+ different object Shapes.
processUser({ id: 1, name: "A" });
processUser({ uuid: '1x', name: "A" });
processUser({ id: 1, name: "A", age: 30 });
processUser({ name: "A", id: 1 }); // DIFFERENT SHAPE (Order matters!)
```

### 1.4 Array Optimizations (ElementsKinde)
V8 categorizes Arrays at the C++ level to optimize storage.
- **Types**: `Smi` (Small Integers - 31 bit), `Double` (Floating point), and `Regular` (Mixed Objects/Strings).
- **Packed vs. Holey**: Arrays without gaps are **Packed**. Arrays with missing indices (e.g., `arr[10] = 1`) are **Holey**.
- **Rule**: Once an array becomes "Holey" or transitions from `Smi` to `Double` or `Regular`, it **never** goes back. 
- Avoid allocating large arrays and pushing to them dynamically. Avoid using `new Array(size_without_fill)` as it initializes a Holey array.

```javascript
// ✅ GOOD: Packed Smi array (Fastest)
const arr = [1, 2, 3]; 

// ❌ BAD: Becomes a Holey Array (Slower prototype chain lookups required)
arr[5] = 6; 

// ❌ BAD: Transitions to Packed Double, then Packed Regular. 
const mixed = [1, 2, 3.5, "text"];
```

---

## 2. Memory Management & Garbage Collection (Orinoco)

V8's Garbage Collector (Orinoco) is Generational. Memory is split into the **Young Generation** and **Old Generation**.

### 2.1 Generational GC
- **Young Generation (Scavenge)**: Uses a fast Semi-Space algorithm. "Minor GCs" occur frequently here. Objects that survive two cycles are "tenured" (moved to the Old Generation).
- **Old Generation (Mark-Sweep-Compact)**: Collects long-lived objects. "Major GC" traverses the entire heap graph starting from GC Roots. It reclaims memory and compacts it to prevent fragmentation. This is "stop-the-world" and blocks the main thread.

### 2.2 Memory Leaks in Modern JS
- **Detached DOM Nodes**: If you remove a DOM node from the document but retain a JavaScript reference to it (or any of its children), the entire tree cannot be collected.
- **Closures**: Variables captured in closures stay in memory as long as the closure is alive. V8 is smart, but holding large objects in setInterval/event listeners leads to massive leaks.
- **WeakMap / WeakSet**: Use these heavily for Caching and DOM bindings. If the key object is unreferenced elsewhere, the Key/Value pair is automatically collected, preventing leaks.

### 2.3 Allocation Profiling
Minimize Minor GC pressure by avoiding unnecessary object creation in hot loops. Reuse objects (Object Pooling) if you are running physics computations, heavy animations, or complex data transformations.

---

## 3. The Event Loop & Concurrency (Advanced)

JavaScript execution is single-threaded but runs concurrently via the Event Loop.

### 3.1 Microtasks vs. Macrotasks
- **Microtasks (Promises, MutationObserver, queueMicrotask)**: Have the highest priority. The Event Loop will exhaust the *entire* Microtask queue before rendering the UI or moving to the next cycle.
  - **Risk**: An infinite loop of Microtasks (a Promise resolving another Promise continuously) will **freeze the main thread** and block the UI entirely.
- **Macrotasks (setTimeout, setInterval, I/O, UI rendering)**: Handled one per cycle. The Event Loop yields to the browser rendering between Macrotasks. Infinite Macrotasks do not freeze the UI, they just lag it.

### 3.2 Node.js Libuv Integration
In Node.js, asynchronous I/O (Crypto, File System, DNS) is offloaded to the C++ **Libuv Thread Pool** (default 4 threads). Once Libuv completes the task, it pushes the callback into the Event Loop's Macrotask queue. 
- **Rule**: Never run synchronous crypto (`crypto.pbkdf2Sync`) or heavy regex strings on the main thread. It starves the Event Loop, blocking all incoming HTTP requests.

---

## 4. World-Class Standards (Human-Centric)

Writing code that TurboFan can optimize is half the battle. The other half is ensuring humans can scale and maintain it.

### 4.1 Google & Airbnb Synthesis
- **Google Standard**: Performance first, strict typing, explicit dependencies, no global state.
- **Airbnb Standard**: Readability, ES6+ syntax utilization, immutability, explicit destructuring.
- **Synthesis Rule**: Prefer immutability and readability, but use mutability (e.g., standard `for` loops over `.reduce()`) in deeply nested, performance-critical "hot paths".

### 4.2 Clean Code for JS
- **Small functions**: Functions should do exactly one thing. If it spans > 25 lines, split it.
- **High Cohesion, Low Coupling**: Modules should have strongly related internals (Cohesion) but rely on minimal external dependencies or knowledge (Coupling).
- **Early Returns**: Avoid `else` blocks whenever possible. Eject from the function as early as possible.

### 4.3 TypeScript at Scale
TypeScript is essentially a compile-time test suite. Do not treat it as just documentational.
- **`infer`**: Use conditional types with `infer` to extract deeply nested return types dynamically rather than hardcoding them.
- **`satisfies`**: Validate an object matches a type without widening its literal types.
- **Template Literal Types**: Enforce exact string patterns natively (e.g., `type Route = \`/api/v1/\${string}\``).
- **Opaque Types (Branded Types)**: Prevent passing raw Primitives (like a generic `string`) into domain-specific IDs. Use `type UserId = string & { readonly __brand: unique symbol }`.

---

## 5. The "Golden Rules" for Monomorphic & Fast JS

Follow this checklist relentlessly.

1. **Initialize All Properties in Constructor**: Explicitly declare every property an object will ever have in the constructor, assigning `null` or `undefined` if data isn't available yet.
2. **Never Change Variable Types**: If a variable starts as an integer, it stays an integer. Do not reassign a boolean memory space to an object.
3. **Always Ensure Consistent Object Keys Order**: `{ a: 1, b: 2 }` and `{ b: 2, a: 1 }` create two distinct Hidden Classes. Always construct objects in exactly the same key order.
4. **Never Use `delete`**: Emulate deletion by setting the value to `undefined` or `null`.
5. **Keep Functions Monomorphic**: Do not write overly generic functions that accept Objects, Strings, and Numbers interchangeably. Write specific handlers for each type.
6. **Pass Arguments Consistently**: Do not call the same function with varied numbers of arguments. Pass `undefined` for omitted arguments.
7. **Avoid Out-of-Bounds Array Access**: Never read past the length of an array. It forces V8 to do an expensive prototype chain lookup on `Array.prototype`.
8. **Avoid Holey Arrays**: Never use `new Array(100)`. If you must pre-allocate, fill it immediately: `new Array(100).fill(0)`.
9. **Don't Mutate Arguments**: Reassigning function parameters (e.g., `function(req){ req.body = 'new'; }`) triggers deoptimizations in strict mode and legacy V8 compilation.
10. **Centralize Regex Patterns**: Compile RegEx strings outside of function bodies to prevent constant GC pressure and recompilation on every invocation.
11. **Always Use WeakMap/WeakSet for DOM References**: Prevent GC memory leaks when tracking state linked to HTML Elements.
12. **Offload Heavy Work to Web Workers**: If a pure JS calculation takes >50ms, the main thread is starving. Move it to a Web Worker (or `worker_threads` in Node). 
13. **Avoid Megamorphic Libraries**: If a utility strictly manipulates objects using generic `Object.keys()` iterations (like deep cloning dynamically), be aware it's megamorphic and very slow.
14. **Always Prefer `const` over `let`**: `const` enforces reference immutability, but V8 heavily relies on it to make static analysis and inlining inferences.
15. **Limit Closure Scope**: Don't capture large arrays or objects inside event listeners. Pass precisely the minimal scalar variables needed.
