# Chapter 2: The Type System You Already Know

JavaScript has types. You've been working with them the entire time. `typeof 42 === 'number'`. `Array.isArray([])`. `null`, `undefined`, objects, functions—these are all types.

TypeScript didn't invent types. It made them **explicit and enforced at compile time** instead of implicit and checked at runtime.

## The Primitive Types

You already know these. TypeScript just names them.

### JavaScript vs TypeScript

```javascript
// JavaScript - types exist, but only at runtime
const name = 'Alice';
const age = 30;
const isActive = true;
const nothing = null;
const notDefined = undefined;

typeof name;      // 'string'
typeof age;       // 'number'
typeof isActive;  // 'boolean'
typeof nothing;   // 'object' (JavaScript's famous quirk)
typeof notDefined; // 'undefined'
```

```typescript
// TypeScript - types are explicit and checked at compile time
const name: string = 'Alice';
const age: number = 30;
const isActive: boolean = true;
const nothing: null = null;
const notDefined: undefined = undefined;
```

But here's the thing: **you rarely need to write those annotations.** TypeScript infers them.

```typescript
const name = 'Alice';        // TypeScript knows this is string
const age = 30;              // TypeScript knows this is number
const isActive = true;       // TypeScript knows this is boolean

// Hover over these in VS Code and you'll see the inferred types
```

Type inference is TypeScript's superpower. The compiler is smart enough to figure out most types from context.

### When Inference Fails (and you need annotations)

```typescript
let x;           // Type: any (TypeScript doesn't know what this will be)
x = 42;          // Still any
x = 'hello';     // Still any, no error

let y: number;   // Type: number (even though uninitialized)
y = 42;          // OK
y = 'hello';     // Error: Type 'string' is not assignable to type 'number'
```

Rule of thumb: **Let TypeScript infer when possible. Annotate when you must.**

## Arrays and Objects

### Arrays

JavaScript arrays can hold anything. TypeScript arrays have opinions.

```typescript
// Inferred as number[]
const numbers = [1, 2, 3];
numbers.push(4);      // OK
numbers.push('five'); // Error

// Inferred as string[]
const names = ['Alice', 'Bob'];
names.push('Charlie'); // OK
names.push(42);        // Error

// Explicit annotation
const scores: number[] = [];
scores.push(100); // OK

// Alternative syntax (same meaning)
const grades: Array<number> = [];
```

Mixed arrays? TypeScript handles them with union types (we'll cover unions properly in Chapter 7, but here's a preview):

```typescript
// Array can hold numbers OR strings
const mixed: (number | string)[] = [1, 'two', 3, 'four'];
mixed.push(5);       // OK
mixed.push('six');   // OK
mixed.push(true);    // Error: boolean not allowed
```

### Objects

JavaScript objects are bags of properties. TypeScript wants to know what's in the bag.

```javascript
// JavaScript - anything goes
const user = {
  name: 'Alice',
  age: 30
};

user.email = 'alice@example.com'; // Surprise! A new property appears
```

```typescript
// TypeScript - infers shape from initialization
const user = {
  name: 'Alice',
  age: 30
};

// TypeScript knows user has name (string) and age (number)
user.name.toUpperCase(); // OK
user.age.toFixed(2);     // OK

user.email = 'alice@example.com';
// Error: Property 'email' does not exist on type '{ name: string; age: number; }'
```

Wait, you can't add properties? Not quite. You can't add properties **the compiler doesn't know about.**

```typescript
// Explicit type allows exactly these properties
const user: { name: string; age: number; email?: string } = {
  name: 'Alice',
  age: 30
};

user.email = 'alice@example.com'; // OK now (email is optional, note the ?)
```

The `?` means optional. We'll dig deeper into object types in Chapter 5.

## Functions

Functions in JavaScript can be called with anything. TypeScript wants to know what goes in and what comes out.

```javascript
// JavaScript
function add(a, b) {
  return a + b;
}

add(2, 3);        // 5
add('hello', ' world'); // 'hello world'
add(2, 'three');  // '2three' (type coercion surprise!)
```

```typescript
// TypeScript
function add(a: number, b: number): number {
  return a + b;
}

add(2, 3);        // 5
add('hello', ' world'); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
add(2, 'three');  // Error
```

The return type (`: number` after the parameters) is often inferred:

```typescript
function add(a: number, b: number) {  // Return type inferred as number
  return a + b;
}
```

Arrow functions work the same way:

```typescript
const add = (a: number, b: number): number => a + b;

// Or with inferred return type
const add = (a: number, b: number) => a + b;
```

Functions get a whole chapter (Chapter 4) because they're where TypeScript really shines.

## The `any` Type (and why you should avoid it)

`any` is TypeScript's escape hatch. It means "I don't know (or care) what type this is."

```typescript
let anything: any = 42;
anything = 'hello';
anything = { name: 'Alice' };
anything.whatever.you.want.nothing.will.error; // No complaints from TypeScript
```

`any` disables type checking. It's useful for:
- Migrating JavaScript to TypeScript incrementally
- Interfacing with untyped libraries
- Genuinely dynamic code where types can't be known

But overusing `any` defeats the purpose of TypeScript. If everything is `any`, you're just writing JavaScript with extra steps.

**Better alternatives to `any`:**

```typescript
// unknown - safer than any, requires type checking before use
let value: unknown = getSomeValue();

if (typeof value === 'string') {
  value.toUpperCase(); // OK, TypeScript knows it's a string here
}

// never - for things that never happen (exhaustiveness checking)
function throwError(message: string): never {
  throw new Error(message);
  // This function never returns normally
}
```

## The `void` Type

Functions that don't return anything have type `void`:

```typescript
function logMessage(message: string): void {
  console.log(message);
  // No return statement
}

// Inferred as void
function logError(error: Error) {
  console.error(error);
}
```

You rarely need to write `: void` explicitly. TypeScript infers it.

## Literal Types

Here's where TypeScript gets interesting. Types don't have to be broad categories like "string" or "number." They can be **specific values**.

```typescript
// This is not a string, it's specifically 'red'
let color: 'red' = 'red';
color = 'blue'; // Error: Type '"blue"' is not assignable to type '"red"'

// Useful for specific sets of values
type Direction = 'north' | 'south' | 'east' | 'west';

function move(direction: Direction) {
  console.log(`Moving ${direction}`);
}

move('north'); // OK
move('up');    // Error: Argument of type '"up"' is not assignable to parameter of type 'Direction'
```

Literal types + unions = powerful constraints:

```typescript
type Status = 'pending' | 'approved' | 'rejected';
type Port = 80 | 443 | 8080;

let status: Status = 'pending';
status = 'approved'; // OK
status = 'canceled'; // Error

let port: Port = 443;
port = 80;   // OK
port = 3000; // Error
```

This is **way** more useful than it looks. We'll see why in Chapter 5.

## Union Types (a preview)

Sometimes a value can be one of several types:

```typescript
// Can be string OR number
let id: string | number;

id = 'abc-123'; // OK
id = 12345;     // OK
id = true;      // Error

// Common pattern: handling values that might not exist
function getUser(id: string | number): User | null {
  const user = database.find(id);
  return user || null;
}

const user = getUser(123);
// user is User | null, must check before using
if (user) {
  console.log(user.name); // OK, TypeScript knows user is not null here
}
```

Unions are fundamental to TypeScript. We'll cover them properly in Chapter 7.

## Type Aliases and Interfaces (a preview)

You can name types for reuse:

```typescript
// Type alias
type UserID = string | number;
type Point = { x: number; y: number };

let id: UserID = 123;
let point: Point = { x: 10, y: 20 };

// Interface (similar, but different - covered in Chapter 5)
interface User {
  id: UserID;
  name: string;
  email: string;
}

const user: User = {
  id: 'abc-123',
  name: 'Alice',
  email: 'alice@example.com'
};
```

When do you use `type` vs `interface`? Chapter 5 has opinions.

## Type Assertions (casting, sort of)

Sometimes you know more than TypeScript does:

```typescript
// TypeScript infers this as Element | null
const input = document.getElementById('username');

// You know it's an HTMLInputElement
const input = document.getElementById('username') as HTMLInputElement;
input.value = 'Alice'; // OK, value property exists on HTMLInputElement

// Alternative syntax (same thing, avoid in React/JSX because of syntax clash)
const input = <HTMLInputElement>document.getElementById('username');
```

Type assertions don't change runtime behavior. They're compile-time instructions: "Trust me, I know this is actually this type."

**Danger:** Type assertions can lie. If you're wrong, you'll get runtime errors.

```typescript
const value = 'hello' as any as number;
value.toFixed(2); // Compiles fine, crashes at runtime
```

Use assertions sparingly. Usually, type guards (checking types at runtime) are safer:

```typescript
const input = document.getElementById('username');

if (input instanceof HTMLInputElement) {
  input.value = 'Alice'; // TypeScript knows it's HTMLInputElement here
}
```

## Null and Undefined

JavaScript has two "nothing" values. TypeScript preserves this distinction.

```typescript
let name: string = 'Alice';
name = null;      // Error (by default in strict mode)
name = undefined; // Error (by default in strict mode)

// Allow null explicitly
let name: string | null = 'Alice';
name = null; // OK now

// Allow undefined explicitly
let name: string | undefined = 'Alice';
name = undefined; // OK now

// Allow both
let name: string | null | undefined = 'Alice';
```

**Strict mode** (which you should use) requires explicit null/undefined handling. This catches the "cannot read property of undefined" errors that plague JavaScript.

```typescript
function greet(name: string | null) {
  console.log(`Hello, ${name.toUpperCase()}`);
  // Error: Object is possibly 'null'

  // Must check first
  if (name) {
    console.log(`Hello, ${name.toUpperCase()}`); // OK
  }
}
```

## The Difference Between Runtime and Compile Time

Critical concept: **TypeScript types exist only at compile time. They disappear in the generated JavaScript.**

```typescript
// TypeScript
function add(a: number, b: number): number {
  return a + b;
}
```

Compiles to:

```javascript
// JavaScript (types removed)
function add(a, b) {
  return a + b;
}
```

This means:
- Type checks don't affect runtime performance (they don't exist at runtime)
- You can't check types with `typeof` or `instanceof` unless they're real JavaScript constructs
- TypeScript can't save you from external data (API responses, user input) without runtime validation

```typescript
interface User {
  name: string;
  age: number;
}

// This is NOT a runtime check
const user: User = JSON.parse(apiResponse);
// If apiResponse is '{"name": "Alice"}', user.age will be undefined at runtime
// TypeScript can't help here because JSON.parse returns 'any'

// Runtime validation is still your job
function isUser(obj: any): obj is User {
  return typeof obj.name === 'string' && typeof obj.age === 'number';
}

const data = JSON.parse(apiResponse);
if (isUser(data)) {
  // Now TypeScript knows data is User
  console.log(data.name, data.age);
}
```

Libraries like Zod, io-ts, and AJV exist specifically to bridge this gap (runtime validation that generates TypeScript types).

## Type Inference Is Your Friend

The biggest mistake beginners make: over-annotating.

**Bad:**

```typescript
const name: string = 'Alice';
const age: number = 30;
const isActive: boolean = true;

function add(a: number, b: number): number {
  const result: number = a + b;
  return result;
}
```

**Good:**

```typescript
const name = 'Alice';        // Inferred as string
const age = 30;              // Inferred as number
const isActive = true;       // Inferred as boolean

function add(a: number, b: number) {  // Return type inferred
  const result = a + b;      // Inferred as number
  return result;
}
```

**When to annotate:**
- Function parameters (can't be inferred)
- Function return types when you want to enforce a contract
- Variables where initialization is separate from declaration
- When inference is wrong or too broad

## Structural Typing (Duck Typing)

TypeScript doesn't care about names. It cares about **shapes**.

```typescript
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

// No explicit Point declaration needed
const point = { x: 10, y: 20 };
logPoint(point); // OK, it has x and y

// Even this works
logPoint({ x: 0, y: 0, z: 0 }); // OK, extra properties don't matter

// But not this
logPoint({ x: 10 }); // Error: Property 'y' is missing
```

If it walks like a duck and quacks like a duck, TypeScript treats it as a duck.

This is **structural typing**. Contrast with **nominal typing** (Java, C#) where you must explicitly declare "this is a Duck."

Structural typing makes TypeScript flexible but can surprise you:

```typescript
interface Config {
  timeout: number;
}

function setup(config: Config) {
  // ...
}

// Typo in property name - still compiles!
setup({ timeOut: 5000 });
// Error: Object literal may only specify known properties, and 'timeOut' does not exist in type 'Config'
```

Wait, that *did* error. TypeScript has a special rule: **excess property checking** for object literals. But it doesn't apply to variables:

```typescript
const config = { timeOut: 5000 }; // No error here (just an object with timeOut)
setup(config); // Error: Type '{ timeOut: number; }' has no properties in common with type 'Config'
```

Actually, this *does* error because the object has no properties that match `Config`. TypeScript's structural typing requires at least some overlap. The difference is that the error message is less helpful than with object literals.

This is a gotcha we'll revisit in Chapter 5.

## What You've Learned

You already knew JavaScript types. Now you know how TypeScript represents them:

- **Primitives:** `string`, `number`, `boolean`, `null`, `undefined`
- **Complex types:** arrays, objects, functions
- **Special types:** `any`, `unknown`, `void`, `never`
- **Literal types:** specific values as types
- **Union types:** value can be one of several types
- **Type inference:** let TypeScript figure it out
- **Structural typing:** shape matters, not name

TypeScript's type system is **gradual**. You can add types incrementally. You can opt out with `any`. You can start strict or start loose and tighten over time.

The goal isn't to type everything. It's to **type enough that the compiler catches meaningful errors** while staying out of your way.

## Next Up

We've covered the types. Now let's talk about the tools that check them.

**Next:** [Chapter 3: Your TypeScript Toolchain](./03-toolchain.md)
