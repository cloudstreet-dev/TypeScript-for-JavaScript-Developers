---
layout: chapter
title: "Functions: Where Things Get Interesting"
chapter_number: 4
permalink: /chapters/functions/
---

# Chapter 4: Functions: Where Things Get Interesting

Functions are the heart of JavaScript. You pass them around, return them, nest them, curry them, compose them. JavaScript functions are flexible, first-class citizens.

TypeScript doesn't change that. It just makes you explicit about **what goes in** and **what comes out**.

This is where TypeScript starts earning its keep.

## Basic Function Types

You've seen this already:

```typescript
function add(a: number, b: number): number {
  return a + b;
}
```

Parameters are annotated. Return type is (optionally) annotated. TypeScript infers the return type if you omit it:

```typescript
function add(a: number, b: number) {
  return a + b; // Inferred return type: number
}
```

**Rule:** Annotate parameters. Let TypeScript infer return types unless you want to enforce a contract.

### Arrow Functions

Same rules apply:

```typescript
const add = (a: number, b: number): number => a + b;

// Inferred return type
const add = (a: number, b: number) => a + b;
```

### Function Expressions

```typescript
const greet: (name: string) => string = (name) => {
  return `Hello, ${name}`;
};
```

The type `(name: string) => string` describes a function that:
- Takes a `string` parameter named `name`
- Returns a `string`

Usually, you don't need this verbosity. Let inference work:

```typescript
const greet = (name: string) => `Hello, ${name}`;
```

## Optional Parameters

JavaScript lets you call functions with missing arguments. TypeScript makes you explicit about it.

```javascript
// JavaScript
function greet(name, greeting) {
  greeting = greeting || 'Hello';
  return `${greeting}, ${name}`;
}

greet('Alice');           // "Hello, Alice"
greet('Alice', 'Hi');     // "Hi, Alice"
```

```typescript
// TypeScript
function greet(name: string, greeting?: string): string {
  greeting = greeting || 'Hello';
  return `${greeting}, ${name}`;
}

greet('Alice');           // OK
greet('Alice', 'Hi');     // OK
greet('Alice', undefined); // OK
```

The `?` marks a parameter as optional. Optional parameters:
- Must come after required parameters
- Have type `T | undefined` (e.g., `string | undefined`)

### Default Parameters

Better than optional: use defaults.

```typescript
function greet(name: string, greeting: string = 'Hello'): string {
  return `${greeting}, ${name}`;
}

greet('Alice');       // "Hello, Alice"
greet('Alice', 'Hi'); // "Hi, Alice"
```

Default parameters are automatically optional. TypeScript infers the type from the default value:

```typescript
function greet(name: string, greeting = 'Hello') {
  // greeting is inferred as string
  return `${greeting}, ${name}`;
}
```

## Rest Parameters

JavaScript's `...rest` syntax works as expected:

```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);       // 6
sum(1, 2, 3, 4, 5); // 15
```

The type `...numbers: number[]` means "zero or more numbers."

You can mix regular and rest parameters:

```typescript
function logWithPrefix(prefix: string, ...messages: string[]): void {
  messages.forEach(msg => console.log(`${prefix}: ${msg}`));
}

logWithPrefix('INFO', 'Server started', 'Port 3000');
// INFO: Server started
// INFO: Port 3000
```

## Function Overloads

JavaScript functions can behave differently based on arguments. TypeScript lets you describe this with **overloads**.

```typescript
// Overload signatures
function parseValue(value: string): string;
function parseValue(value: number): number;
function parseValue(value: boolean): boolean;

// Implementation signature (must be compatible with all overloads)
function parseValue(value: string | number | boolean): string | number | boolean {
  if (typeof value === 'string') {
    return value.trim();
  } else if (typeof value === 'number') {
    return value * 2;
  } else {
    return !value;
  }
}

const a = parseValue('  hello  '); // Type: string
const b = parseValue(42);           // Type: number
const c = parseValue(true);         // Type: boolean
```

Overloads let you narrow return types based on input types.

**When to use overloads:**
- Different input types produce different output types
- Different numbers/combinations of parameters produce different outputs

**When NOT to use overloads:**
- You can express it with union types or generics (we'll cover generics in Chapter 6)

### Real-World Example: `createElement`

DOM's `createElement` returns different types based on the tag:

```typescript
function createElement(tag: 'div'): HTMLDivElement;
function createElement(tag: 'span'): HTMLSpanElement;
function createElement(tag: 'canvas'): HTMLCanvasElement;
function createElement(tag: string): HTMLElement;

function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}

const div = createElement('div');       // Type: HTMLDivElement
const span = createElement('span');     // Type: HTMLSpanElement
const custom = createElement('my-tag'); // Type: HTMLElement
```

Now `div` has `HTMLDivElement`-specific properties. TypeScript knows.

## `this` in Functions

JavaScript's `this` is famously confusing. TypeScript lets you annotate it.

```typescript
interface User {
  name: string;
  greet(this: User): void;
}

const user: User = {
  name: 'Alice',
  greet() {
    console.log(`Hello, ${this.name}`);
  }
};

user.greet(); // OK

const greet = user.greet;
greet(); // Error: The 'this' context of type 'void' is not assignable to method's 'this' of type 'User'
```

The `this: User` parameter is a **fake first parameter** (it doesn't exist at runtime). It tells TypeScript what `this` should be.

**In practice:** You rarely need this. Arrow functions and class methods usually handle it. But it's there when you need it.

## Callbacks and Higher-Order Functions

Functions that take functions? TypeScript loves those.

```typescript
function map<T, U>(array: T[], fn: (item: T) => U): U[] {
  return array.map(fn);
}

const numbers = [1, 2, 3];
const doubled = map(numbers, n => n * 2); // Type: number[]
const strings = map(numbers, n => n.toString()); // Type: string[]
```

Don't panic about the `<T, U>` yet. That's generics (Chapter 6). Focus on the callback:

```typescript
fn: (item: T) => U
```

This says: `fn` is a function that takes a `T` and returns a `U`.

### Real-World Callback Example

```typescript
type EventCallback = (event: Event) => void;

function addEventListener(element: HTMLElement, event: string, callback: EventCallback): void {
  element.addEventListener(event, callback);
}

const button = document.querySelector('button')!;
addEventListener(button, 'click', (event) => {
  console.log('Button clicked', event);
});
```

The callback type is explicit. TypeScript knows `event` is an `Event`.

## Async Functions

Async functions return Promises. TypeScript knows this.

```typescript
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return data; // Must be User
}

// Inferred return type: Promise<User>
```

If you omit `Promise<User>`, TypeScript infers it. But annotating it enforces the contract—the function **must** return a Promise that resolves to a User.

### Error Handling in Async Functions

```typescript
async function fetchUser(id: string): Promise<User | null> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return null;
    }
    return await response.json();
  } catch (error) {
    console.error(error);
    return null;
  }
}
```

The return type is `Promise<User | null>`. Callers must handle the null case.

### Async Callbacks

```typescript
type AsyncCallback<T> = (value: T) => Promise<void>;

async function processItems<T>(items: T[], callback: AsyncCallback<T>): Promise<void> {
  for (const item of items) {
    await callback(item);
  }
}

const ids = [1, 2, 3];
await processItems(ids, async (id) => {
  const user = await fetchUser(id.toString());
  console.log(user);
});
```

Callbacks can be async. TypeScript handles the Promise types.

## Void vs Undefined vs Never

Three "nothing" return types. They mean different things.

### `void` - Function doesn't return a useful value

```typescript
function logMessage(message: string): void {
  console.log(message);
}
```

`void` means "this function's return value doesn't matter." You can technically return `undefined` from a void function (JavaScript functions return `undefined` by default), but the caller shouldn't care.

**Key point:** You can assign any function to a `void`-returning function:

```typescript
type VoidFunc = () => void;

const f: VoidFunc = () => 42; // OK, return value ignored

const result = f(); // Type: void (even though it actually returns 42)
```

This is by design. Callbacks often have `void` return types, but you can pass any function.

### `undefined` - Function explicitly returns undefined

```typescript
function doNothing(): undefined {
  return undefined;
}
```

The function **must** return `undefined`. Can't omit the return.

**Rarely used.** Usually `void` is better.

### `never` - Function never returns

```typescript
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // ...
  }
}
```

`never` is for functions that:
- Throw exceptions
- Have infinite loops
- Call `process.exit()` or similar

The function doesn't just return nothing—it **never returns at all**.

### When it matters

```typescript
function handleValue(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase();
  } else if (typeof value === 'number') {
    return value.toFixed(2);
  } else {
    // TypeScript knows this is unreachable
    const exhaustive: never = value;
    throw new Error(`Unhandled value: ${exhaustive}`);
  }
}
```

If you add a new type to the union but forget to handle it, the `never` assignment will error. Exhaustiveness checking (we'll see more in Chapter 7).

## Function Type Aliases

You can name function types for reuse:

```typescript
type BinaryOp = (a: number, b: number) => number;

const add: BinaryOp = (a, b) => a + b;
const subtract: BinaryOp = (a, b) => a - b;
const multiply: BinaryOp = (a, b) => a * b;

function applyOperation(a: number, b: number, op: BinaryOp): number {
  return op(a, b);
}

applyOperation(10, 5, add);      // 15
applyOperation(10, 5, subtract); // 5
```

Common in React for event handlers:

```typescript
type ClickHandler = (event: React.MouseEvent) => void;

const handleClick: ClickHandler = (event) => {
  console.log(event.clientX, event.clientY);
};
```

## Function Interfaces

Interfaces can describe functions (though type aliases are more common for this):

```typescript
interface SearchFunc {
  (query: string, limit: number): string[];
}

const search: SearchFunc = (query, limit) => {
  // Implementation
  return [];
};
```

But interfaces can also describe callable objects with properties:

```typescript
interface Counter {
  (start: number): void;
  interval: number;
  reset(): void;
}

function createCounter(): Counter {
  const counter = function(start: number) {
    console.log(`Starting from ${start}`);
  } as Counter;

  counter.interval = 1000;
  counter.reset = () => console.log('Reset!');

  return counter;
}

const counter = createCounter();
counter(10);         // Calling the function
counter.reset();     // Calling a method
console.log(counter.interval); // Accessing a property
```

**Rare in practice.** Most functions are just functions.

## Practical Patterns

### Type Guards

Functions that narrow types:

```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function processValue(value: unknown) {
  if (isString(value)) {
    // TypeScript knows value is string here
    console.log(value.toUpperCase());
  }
}
```

`value is string` is a **type predicate**. It tells TypeScript that if the function returns true, the parameter has that type.

### Assertion Functions

Similar to type guards, but for assertions:

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error('Not a string');
  }
}

function processValue(value: unknown) {
  assertIsString(value);
  // If we reach here, value is definitely string
  console.log(value.toUpperCase());
}
```

The `asserts value is string` syntax tells TypeScript that if the function doesn't throw, the value is that type.

### Factory Functions

Functions that return objects:

```typescript
interface User {
  id: string;
  name: string;
  createdAt: Date;
}

function createUser(name: string): User {
  return {
    id: generateId(),
    name,
    createdAt: new Date()
  };
}

const user = createUser('Alice'); // Type: User
```

TypeScript enforces that the returned object matches the `User` interface.

### Builder Pattern

Chainable methods:

```typescript
class QueryBuilder {
  private conditions: string[] = [];

  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }

  orderBy(field: string): this {
    // ...
    return this;
  }

  build(): string {
    return this.conditions.join(' AND ');
  }
}

const query = new QueryBuilder()
  .where('age > 18')
  .where('active = true')
  .orderBy('name')
  .build();
```

Returning `this` maintains type for chaining.

### Currying

```typescript
function add(a: number): (b: number) => number {
  return (b) => a + b;
}

const add5 = add(5);
console.log(add5(10)); // 15

// Or inline
console.log(add(5)(10)); // 15
```

TypeScript infers the return type from the nested function.

### Partial Application

```typescript
function fetchWithDefaults(url: string, options: RequestInit): Promise<Response> {
  return fetch(url, options);
}

function fetchJSON(url: string): Promise<any> {
  return fetchWithDefaults(url, {
    headers: { 'Content-Type': 'application/json' }
  }).then(r => r.json());
}
```

Pre-fill some arguments, return a new function. Note: We use `fetchWithDefaults` to avoid shadowing the global `fetch`.

## Common Pitfalls

### Forgetting to Annotate Parameters

```typescript
// Bad: parameters are 'any'
function add(a, b) {
  return a + b;
}

// Good
function add(a: number, b: number) {
  return a + b;
}
```

If you forget parameter types, they default to `any` (unless `noImplicitAny` is enabled, which it should be).

### Over-Annotating Return Types

```typescript
// Unnecessary
function add(a: number, b: number): number {
  return a + b;
}

// Simpler
function add(a: number, b: number) {
  return a + b;
}
```

Let TypeScript infer return types unless:
- You want to enforce a specific return type (contract)
- The inferred type is too broad or complex
- You're writing a public API

### Misunderstanding `void`

```typescript
type Callback = () => void;

const callback: Callback = () => {
  return 42; // This is OK!
};

const result = callback(); // Type: void
```

`void` doesn't mean "must not return anything." It means "the return value doesn't matter."

### Using `Function` Type

```typescript
// Bad: 'Function' is too loose
function runCallback(callback: Function) {
  callback();
}

// Good: be specific
function runCallback(callback: () => void) {
  callback();
}
```

`Function` is a global type that accepts any function. Always specify the signature.

## What You've Learned

- **Parameter types** are required. Return types are usually inferred.
- **Optional parameters** use `?`. Default parameters are implicitly optional.
- **Rest parameters** use `...` and have array types.
- **Overloads** let you define multiple signatures for the same function.
- **`this`** can be annotated as a fake first parameter.
- **Callbacks** are just function-typed parameters.
- **Async functions** return `Promise<T>`.
- **`void`** means "return value doesn't matter."
- **`never`** means "this function never returns."
- **Type guards** narrow types at runtime.
- **Let TypeScript infer** return types unless you need to enforce a contract.

Functions are where TypeScript really starts protecting you. Parameter types catch incorrect calls. Return types ensure consistency. Overloads and generics (next chapters) make complex APIs type-safe.

---

**Next:** [Chapter 5: Interfaces, Types, and the Art of Shapes](./05-interfaces-types.md)
