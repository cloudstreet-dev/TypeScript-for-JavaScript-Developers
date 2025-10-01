# Chapter 5: Interfaces, Types, and the Art of Shapes

JavaScript objects are flexible. You can add properties, delete them, pass them around, and duck-type your way through any situation.

TypeScript preserves that flexibility while adding one crucial layer: **shape checking**. Before you pass an object somewhere, TypeScript verifies it has the right properties with the right types.

This chapter is about defining those shapes.

## Interfaces: Describing Object Shapes

An interface defines the structure of an object:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

const user: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
};
```

Simple. The object must have `id`, `name`, and `email`, all strings.

### Optional Properties

Not all properties are required:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;  // Optional
}

const user1: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
  // age is missing - OK
};

const user2: User = {
  id: '456',
  name: 'Bob',
  email: 'bob@example.com',
  age: 30
  // age is present - also OK
};
```

Optional properties have type `T | undefined`. You must check before using:

```typescript
function greetUser(user: User) {
  console.log(`Hello, ${user.name}`);

  if (user.age) {
    console.log(`You are ${user.age} years old`);
  }
}
```

### Readonly Properties

Some properties shouldn't change after creation:

```typescript
interface User {
  readonly id: string;
  name: string;
  email: string;
}

const user: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
};

user.name = 'Alicia'; // OK
user.id = '456';      // Error: Cannot assign to 'id' because it is a read-only property
```

`readonly` is compile-time only. At runtime, JavaScript has no such restriction. But TypeScript won't let you modify it.

### Index Signatures

Sometimes you don't know all property names ahead of time:

```typescript
interface Dictionary {
  [key: string]: number;
}

const scores: Dictionary = {
  alice: 95,
  bob: 87,
  charlie: 92
};

scores.david = 88; // OK
scores.eve = 'high'; // Error: Type 'string' is not assignable to type 'number'
```

`[key: string]: number` means "any string key maps to a number."

You can combine index signatures with known properties:

```typescript
interface Config {
  host: string;
  port: number;
  [key: string]: any; // Additional properties allowed
}

const config: Config = {
  host: 'localhost',
  port: 3000,
  debug: true,
  logLevel: 'info'
};
```

### Extending Interfaces

Interfaces can extend other interfaces:

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: string;
  department: string;
}

const employee: Employee = {
  name: 'Alice',
  age: 30,
  employeeId: 'E123',
  department: 'Engineering'
};
```

Multiple inheritance works too:

```typescript
interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface User extends Person, Timestamped {
  email: string;
}

const user: User = {
  name: 'Alice',
  age: 30,
  email: 'alice@example.com',
  createdAt: new Date(),
  updatedAt: new Date()
};
```

## Type Aliases: Alternative Syntax

Type aliases do similar things but with different syntax:

```typescript
type User = {
  id: string;
  name: string;
  email: string;
};

const user: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
};
```

Looks almost the same. So what's the difference?

## Interfaces vs Type Aliases

### Interfaces Can Be Reopened (Declaration Merging)

```typescript
interface User {
  id: string;
  name: string;
}

// Later, in the same scope
interface User {
  email: string;
}

// User now has id, name, AND email
const user: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
};
```

This is **declaration merging**. Multiple `interface` declarations with the same name merge into one.

Type aliases can't do this:

```typescript
type User = {
  id: string;
  name: string;
};

type User = {  // Error: Duplicate identifier 'User'
  email: string;
};
```

**When this matters:** Library type definitions sometimes use declaration merging to extend types. For your own code, you rarely want this.

### Type Aliases Can Represent More Than Objects

```typescript
// Union type
type Status = 'pending' | 'approved' | 'rejected';

// Intersection type
type Employee = Person & Timestamped;

// Primitive type alias
type ID = string | number;

// Tuple
type Point = [number, number];

// Function type
type BinaryOp = (a: number, b: number) => number;
```

Interfaces are for objects (mostly). Types are for everything.

### Interfaces Show Better Error Messages

When something goes wrong, interfaces often produce clearer errors. Try this:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

const user: User = {
  id: '123',
  name: 'Alice'
  // Missing email
};
// Error: Property 'email' is missing in type '{ id: string; name: string; }' but required in type 'User'.
```

Type aliases sometimes generate verbose, hard-to-read errors (especially with complex types).

### Performance (Barely Matters)

Interfaces are slightly faster to check in some scenarios. In practice, the difference is negligible unless you're defining thousands of types.

### The Verdict

**Use `interface` for objects.** Use `type` for unions, intersections, primitives, tuples, and functions.

**Both work for most cases.** Consistency matters more than the choice.

Some teams prefer `type` everywhere. Some prefer `interface` for objects. Pick a convention and stick to it.

## Intersection Types

Combine multiple types into one:

```typescript
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: string;
  department: string;
};

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
  name: 'Alice',
  age: 30,
  employeeId: 'E123',
  department: 'Engineering'
};
```

`Person & Employee` means "has all properties from both."

Intersections work with interfaces too:

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: string;
  department: string;
}

type EmployeePerson = Person & Employee;
```

## Union Types

A value can be one of several types:

```typescript
type Status = 'pending' | 'approved' | 'rejected';

let status: Status = 'pending';
status = 'approved'; // OK
status = 'denied';   // Error: Type '"denied"' is not assignable to type 'Status'
```

Unions work with objects too:

```typescript
type SuccessResponse = {
  status: 'success';
  data: any;
};

type ErrorResponse = {
  status: 'error';
  message: string;
};

type Response = SuccessResponse | ErrorResponse;

function handleResponse(response: Response) {
  if (response.status === 'success') {
    console.log(response.data);
  } else {
    console.log(response.message);
  }
}
```

TypeScript **narrows** the type based on the `status` check. This is called **discriminated unions** (more on that in Chapter 7).

## Literal Types and Const Assertions

Literal types aren't just primitives. They're **specific values**:

```typescript
type Direction = 'north' | 'south' | 'east' | 'west';

let direction: Direction = 'north';
direction = 'up'; // Error
```

Const assertions lock down types:

```typescript
const config = {
  host: 'localhost',
  port: 3000
};
// Type: { host: string; port: number }

const config = {
  host: 'localhost',
  port: 3000
} as const;
// Type: { readonly host: "localhost"; readonly port: 3000 }
```

With `as const`:
- Properties become `readonly`
- Strings become literal types ("localhost" instead of string)
- Numbers become literal types (3000 instead of number)

Useful for configuration objects and constants.

## Nested Objects

Interfaces and types can nest:

```typescript
interface Address {
  street: string;
  city: string;
  country: string;
}

interface User {
  id: string;
  name: string;
  address: Address;
}

const user: User = {
  id: '123',
  name: 'Alice',
  address: {
    street: '123 Main St',
    city: 'Springfield',
    country: 'USA'
  }
};
```

Or inline:

```typescript
interface User {
  id: string;
  name: string;
  address: {
    street: string;
    city: string;
    country: string;
  };
}
```

Both work. Separate interfaces are better for reusability.

## Arrays in Interfaces

```typescript
interface User {
  id: string;
  name: string;
  roles: string[];
}

const user: User = {
  id: '123',
  name: 'Alice',
  roles: ['admin', 'editor']
};
```

Or use `Array<T>`:

```typescript
interface User {
  id: string;
  name: string;
  roles: Array<string>;
}
```

Same thing. `string[]` is more common.

## Methods in Interfaces

Interfaces can describe object methods:

```typescript
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

const calc: Calculator = {
  add(a, b) {
    return a + b;
  },
  subtract(a, b) {
    return a - b;
  }
};
```

Or using property syntax:

```typescript
interface Calculator {
  add: (a: number, b: number) => number;
  subtract: (a: number, b: number) => number;
}
```

Both describe functions. Method syntax is cleaner.

## Callable Interfaces

Interfaces can describe functions:

```typescript
interface Greeter {
  (name: string): string;
}

const greet: Greeter = (name) => `Hello, ${name}`;

console.log(greet('Alice')); // "Hello, Alice"
```

Useful? Rarely. Type aliases are cleaner for this:

```typescript
type Greeter = (name: string) => string;
```

## Hybrid Types (Functions with Properties)

JavaScript functions can have properties. TypeScript can describe them:

```typescript
interface Counter {
  (start: number): string;
  interval: number;
  reset(): void;
}

function createCounter(): Counter {
  const counter = (function (start: number) {
    return `Starting from ${start}`;
  }) as Counter;

  counter.interval = 1000;
  counter.reset = () => {
    console.log('Counter reset');
  };

  return counter;
}

const counter = createCounter();
console.log(counter(10));    // "Starting from 10"
console.log(counter.interval); // 1000
counter.reset();             // "Counter reset"
```

**Real-world use:** Rare. Libraries sometimes use this pattern (jQuery, Express middleware), but modern code doesn't.

## Structural Typing Revisited

TypeScript doesn't care about names. It cares about **shapes**.

```typescript
interface Point {
  x: number;
  y: number;
}

function distance(p: Point): number {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

const point = { x: 3, y: 4 };
console.log(distance(point)); // OK, shape matches

const vector = { x: 1, y: 2, z: 3 };
console.log(distance(vector)); // OK, extra property doesn't matter
```

But object literals have **excess property checking**:

```typescript
distance({ x: 3, y: 4, z: 5 });
// Error: Object literal may only specify known properties, and 'z' does not exist in type 'Point'
```

TypeScript is stricter with inline literals to catch typos. Assigned variables don't get this check.

**To allow extra properties explicitly:**

```typescript
interface Point {
  x: number;
  y: number;
  [key: string]: any;
}

distance({ x: 3, y: 4, z: 5 }); // OK now
```

## Type Assertions and Casting

Sometimes you know more than TypeScript:

```typescript
const input = document.getElementById('username');
// Type: HTMLElement | null

// Assert it's an HTMLInputElement
const typedInput = input as HTMLInputElement;
typedInput.value = 'Alice';

// Or with angle brackets (avoid in JSX/React)
const typedInput = <HTMLInputElement>input;
```

Assertions don't change runtime behavior. They're compile-time hints.

**Assertions can lie:**

```typescript
const value = 'hello' as any as number;
value.toFixed(2); // Compiles, crashes at runtime
```

Use them sparingly. Prefer type guards (runtime checks):

```typescript
const input = document.getElementById('username');

if (input instanceof HTMLInputElement) {
  input.value = 'Alice'; // TypeScript knows it's HTMLInputElement
}
```

## Utility Types (Sneak Peek)

TypeScript has built-in helpers for transforming types:

### `Partial<T>` - All properties optional

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// Same as: { id?: string; name?: string; email?: string; }

function updateUser(id: string, updates: Partial<User>) {
  // Can pass any subset of User properties
}

updateUser('123', { name: 'Alice' }); // OK
```

### `Required<T>` - All properties required

```typescript
interface Config {
  host?: string;
  port?: number;
}

type RequiredConfig = Required<Config>;
// Same as: { host: string; port: number; }
```

### `Readonly<T>` - All properties readonly

```typescript
interface User {
  id: string;
  name: string;
}

type ReadonlyUser = Readonly<User>;
// Same as: { readonly id: string; readonly name: string; }

const user: ReadonlyUser = { id: '123', name: 'Alice' };
user.name = 'Bob'; // Error: Cannot assign to 'name' because it is a read-only property
```

### `Pick<T, K>` - Select specific properties

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Pick<User, 'id' | 'name' | 'email'>;
// Same as: { id: string; name: string; email: string; }
```

### `Omit<T, K>` - Exclude specific properties

```typescript
type PublicUser = Omit<User, 'password'>;
// Same as: { id: string; name: string; email: string; }
```

We'll cover more utility types in Chapter 7.

## Practical Patterns

### API Response Types

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

interface User {
  id: string;
  name: string;
  email: string;
}

async function fetchUser(id: string): Promise<ApiResponse<User>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    return { success: true, data };
  } catch (error) {
    return { success: false, error: String(error) };
  }
}

const result = await fetchUser('123');
if (result.success && result.data) {
  console.log(result.data.name);
}
```

### Builder Pattern

```typescript
interface QueryOptions {
  limit?: number;
  offset?: number;
  orderBy?: string;
}

class QueryBuilder {
  private options: QueryOptions = {};

  limit(n: number): this {
    this.options.limit = n;
    return this;
  }

  offset(n: number): this {
    this.options.offset = n;
    return this;
  }

  orderBy(field: string): this {
    this.options.orderBy = field;
    return this;
  }

  build(): QueryOptions {
    return this.options;
  }
}

const query = new QueryBuilder()
  .limit(10)
  .offset(20)
  .orderBy('createdAt')
  .build();
```

### Discriminated Unions (Preview)

```typescript
interface SuccessResult {
  status: 'success';
  data: string;
}

interface ErrorResult {
  status: 'error';
  message: string;
}

type Result = SuccessResult | ErrorResult;

function handleResult(result: Result) {
  if (result.status === 'success') {
    console.log(result.data); // TypeScript knows this is SuccessResult
  } else {
    console.log(result.message); // TypeScript knows this is ErrorResult
  }
}
```

The `status` property discriminates between the two types. More on this in Chapter 7.

### Brands (Nominal Typing Simulation)

TypeScript is structural, but sometimes you want nominal typing:

```typescript
// Use unique symbol for proper branding
declare const __userIdBrand: unique symbol;
declare const __postIdBrand: unique symbol;

type UserId = string & { readonly [__userIdBrand]: never };
type PostId = string & { readonly [__postIdBrand]: never };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createPostId(id: string): PostId {
  return id as PostId;
}

function getUserPosts(userId: UserId, postId: PostId) {
  // ...
}

const userId = createUserId('user-123');
const postId = createPostId('post-456');

getUserPosts(userId, postId); // OK
getUserPosts(postId, userId); // Error: Types don't match
```

The `unique symbol` ensures the brands are truly distinct. The brand property doesn't exist at runtime—it's a compile-time trick to make strings distinguishable.

## Common Pitfalls

### Circular References

Recursive types are allowed, but you must handle the base case:

```typescript
interface Node {
  value: number;
  next: Node; // This definition is FINE
}

// But this initialization is impossible:
const node: Node = {
  value: 1,
  next: ??? // Can't satisfy this without null/undefined
};
```

Solution: make it optional or nullable:

```typescript
interface Node {
  value: number;
  next: Node | null;
}

const node: Node = {
  value: 1,
  next: {
    value: 2,
    next: null // Base case
  }
};
```

### Forgetting Optional Chaining

```typescript
interface User {
  profile?: {
    avatar?: string;
  };
}

const user: User = {};
console.log(user.profile.avatar); // Runtime error: Cannot read property 'avatar' of undefined

// Safe:
console.log(user.profile?.avatar);
```

### Index Signatures Swallow Everything

```typescript
interface Config {
  host: string;
  port: number;
  [key: string]: any;
}

const config: Config = {
  host: 'localhost',
  prot: 3000  // Typo! But no error because of index signature
};
```

Index signatures disable typo checking. Use them sparingly.

## What You've Learned

- **Interfaces** describe object shapes
- **Type aliases** can describe anything (objects, unions, primitives, functions)
- **Use `interface` for objects, `type` for everything else** (or pick one and be consistent)
- **Optional properties** use `?`, readonly properties use `readonly`
- **Intersections** (`&`) combine types
- **Unions** (`|`) allow multiple types
- **Literal types** are specific values, not broad categories
- **Index signatures** allow dynamic keys
- **Structural typing** means shape matters, not name
- **Excess property checking** catches typos in object literals
- **Utility types** transform existing types

Interfaces and types are the building blocks of TypeScript's type system. Master these, and you're halfway to mastering TypeScript.

---

**Next:** [Chapter 6: Generics (or: How I Learned to Stop Worrying and Love &lt;T&gt;)](./06-generics.md)
