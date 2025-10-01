---
layout: chapter
title: "Advanced Types and the Compiler's Bag of Tricks"
chapter_number: 7
permalink: /chapters/advanced-types/
---

# Chapter 7: Advanced Types and the Compiler's Bag of Tricks

TypeScript's type system is surprisingly powerful. Beyond basic types and generics lies a layer of advanced features that let you express complex constraints, transform types programmatically, and catch subtle bugs at compile time.

This is where TypeScript graduates from "JavaScript with types" to "a type-level programming language."

## Union Types (Revisited)

You've seen unions. Let's explore their power.

```typescript
type Status = 'pending' | 'approved' | 'rejected';
type ID = string | number;
type Result = Success | Error;
```

Unions represent "one of these types." TypeScript narrows unions through **type guards**.

### Type Narrowing with typeof

```typescript
function processValue(value: string | number) {
  if (typeof value === 'string') {
    // TypeScript knows value is string here
    return value.toUpperCase();
  } else {
    // TypeScript knows value is number here
    return value.toFixed(2);
  }
}
```

The `typeof` check narrows the union.

### Type Narrowing with instanceof

```typescript
class Dog {
  bark() {
    console.log('Woof!');
  }
}

class Cat {
  meow() {
    console.log('Meow!');
  }
}

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

### Type Narrowing with `in`

```typescript
interface Car {
  drive(): void;
}

interface Boat {
  sail(): void;
}

function move(vehicle: Car | Boat) {
  if ('drive' in vehicle) {
    vehicle.drive();
  } else {
    vehicle.sail();
  }
}
```

The `in` operator checks if a property exists, narrowing the type.

### Discriminated Unions (Tagged Unions)

The most powerful pattern for unions:

```typescript
interface SuccessResponse {
  status: 'success';
  data: string;
}

interface ErrorResponse {
  status: 'error';
  message: string;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  if (response.status === 'success') {
    console.log(response.data); // TypeScript knows this is SuccessResponse
  } else {
    console.log(response.message); // TypeScript knows this is ErrorResponse
  }
}
```

The `status` property **discriminates** between the union members. TypeScript uses it to narrow.

**Why this matters:**

```typescript
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

interface Triangle {
  kind: 'triangle';
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    case 'triangle':
      return (shape.base * shape.height) / 2;
    default:
      // Exhaustiveness checking
      const exhaustive: never = shape;
      throw new Error(`Unhandled shape: ${exhaustive}`);
  }
}
```

If you add a new shape but forget to handle it, the `never` assignment will error. **Exhaustiveness checking** catches missing cases at compile time.

## Intersection Types (Revisited)

Intersections combine types:

```typescript
type HasName = { name: string };
type HasAge = { age: number };
type Person = HasName & HasAge;

const person: Person = {
  name: 'Alice',
  age: 30
};
```

Intersections are useful for **mixins**:

```typescript
function withTimestamp<T>(obj: T): T & { timestamp: Date } {
  return { ...obj, timestamp: new Date() };
}

const user = { name: 'Alice' };
const timestampedUser = withTimestamp(user);
// Type: { name: string } & { timestamp: Date }

console.log(timestampedUser.name);      // Alice
console.log(timestampedUser.timestamp); // Date
```

## Mapped Types

Transform one type into another by mapping over its properties.

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

interface User {
  name: string;
  age: number;
}

type ReadonlyUser = Readonly<User>;
// Same as: { readonly name: string; readonly age: number; }
```

Breaking it down:
- `keyof T` - Get all keys of `T` ('name' | 'age')
- `P in keyof T` - Iterate over each key
- `T[P]` - Get the type of that property

### Partial (Make All Properties Optional)

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

function updateUser(id: string, updates: Partial<User>) {
  // Can pass any subset of User properties
}

updateUser('123', { name: 'Alice' }); // OK
updateUser('123', { age: 31 });       // OK
updateUser('123', {});                // OK
```

### Required (Make All Properties Required)

```typescript
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```

The `-?` removes the optional modifier.

### Pick (Select Specific Properties)

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Pick<User, 'id' | 'name' | 'email'>;
// { id: string; name: string; email: string; }
```

### Omit (Exclude Specific Properties)

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

type PublicUser = Omit<User, 'password'>;
// { id: string; name: string; email: string; }
```

### Record (Create Object Type with Specific Keys and Value Type)

```typescript
type Record<K extends keyof any, T> = {
  [P in K]: T;
};

type PageInfo = {
  title: string;
  url: string;
};

type Pages = Record<'home' | 'about' | 'contact', PageInfo>;
// {
//   home: PageInfo;
//   about: PageInfo;
//   contact: PageInfo;
// }

const pages: Pages = {
  home: { title: 'Home', url: '/' },
  about: { title: 'About', url: '/about' },
  contact: { title: 'Contact', url: '/contact' }
};
```

### Custom Mapped Types

```typescript
// Make all properties nullable
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

// Deep readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Make specific properties optional
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface User {
  id: string;
  name: string;
  email: string;
}

type UserWithOptionalEmail = Optional<User, 'email'>;
// { id: string; name: string; email?: string; }
```

## Conditional Types

Types that depend on conditions:

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

The syntax: `T extends U ? X : Y` (like a ternary).

### Extract Non-Nullable Types

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type A = NonNullable<string | null>;      // string
type B = NonNullable<number | undefined>; // number
type C = NonNullable<string | null | undefined>; // string
```

### Extract Function Return Type

```typescript
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : never;

function getUser() {
  return { id: '1', name: 'Alice' };
}

type User = ReturnType<typeof getUser>;
// { id: string; name: string; }
```

The `infer` keyword captures the return type.

### Extract Function Parameters

```typescript
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number) {
  console.log(`Hello, ${name}, you are ${age} years old`);
}

type GreetParams = Parameters<typeof greet>;
// [name: string, age: number]
```

### Awaited (Unwrap Promises)

```typescript
type Awaited<T> = T extends Promise<infer U> ? U : T;

type A = Awaited<Promise<string>>; // string
type B = Awaited<Promise<Promise<number>>>; // Promise<number> (not recursive by default)
```

Built-in version is recursive:

```typescript
async function fetchData() {
  return { id: 1, name: 'Alice' };
}

type Data = Awaited<ReturnType<typeof fetchData>>;
// { id: number; name: string; }
```

## Template Literal Types

Build types from string templates:

```typescript
type Greeting = `Hello, ${string}`;

const g1: Greeting = 'Hello, world'; // OK
const g2: Greeting = 'Hi there';     // Error
```

Combine with unions:

```typescript
type Color = 'red' | 'green' | 'blue';
type Shade = 'light' | 'dark';

type ColorVariant = `${Shade}-${Color}`;
// 'light-red' | 'light-green' | 'light-blue' | 'dark-red' | 'dark-green' | 'dark-blue'
```

### CSS Property Types

```typescript
type CSSProperty = 'color' | 'background' | 'border';
type CSSHoverProperty = `${CSSProperty}:hover`;
// 'color:hover' | 'background:hover' | 'border:hover'
```

### Event Handler Types

```typescript
type EventName = 'click' | 'focus' | 'blur';
type EventHandler = `on${Capitalize<EventName>}`;
// 'onClick' | 'onFocus' | 'onBlur'

interface ButtonProps {
  onClick?: () => void;
  onFocus?: () => void;
  onBlur?: () => void;
}
```

### String Manipulation Utilities

```typescript
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;

type A = Uppercase<'hello'>; // 'HELLO'
type B = Lowercase<'WORLD'>; // 'world'
type C = Capitalize<'typescript'>; // 'Typescript'
type D = Uncapitalize<'Hello'>; // 'hello'
```

## Index Access Types

Extract property types:

```typescript
interface User {
  id: string;
  name: string;
  age: number;
}

type UserName = User['name']; // string
type UserAge = User['age'];   // number
type UserIdOrName = User['id' | 'name']; // string | string → string
```

Useful with generic constraints:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Alice', age: 30 };
const name = getProperty(user, 'name'); // Type: string
const age = getProperty(user, 'age');   // Type: number
```

## Type Guards (Custom)

User-defined type narrowing:

```typescript
interface Cat {
  meow(): void;
}

interface Dog {
  bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return 'meow' in animal;
}

function speak(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow(); // TypeScript knows it's Cat
  } else {
    animal.bark(); // TypeScript knows it's Dog
  }
}
```

The `animal is Cat` syntax is a **type predicate**. It tells TypeScript that if the function returns `true`, the parameter is that type.

### Assertion Functions

```typescript
function assertIsDefined<T>(value: T): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error('Value is null or undefined');
  }
}

function processValue(value: string | null) {
  assertIsDefined(value);
  // TypeScript knows value is string (not null) after this line
  console.log(value.toUpperCase());
}
```

The `asserts` keyword tells TypeScript that if the function doesn't throw, the assertion is true.

## `keyof` and Lookup Types

`keyof` gets the keys of a type:

```typescript
interface User {
  id: string;
  name: string;
  age: number;
}

type UserKey = keyof User; // 'id' | 'name' | 'age'
```

Combine with lookup types:

```typescript
type UserValue = User[keyof User]; // string | number
```

Practical example:

```typescript
function pluck<T, K extends keyof T>(objects: T[], key: K): T[K][] {
  return objects.map(obj => obj[key]);
}

const users = [
  { name: 'Alice', age: 30 },
  { name: 'Bob', age: 25 }
];

const names = pluck(users, 'name'); // string[]
const ages = pluck(users, 'age');   // number[]
```

## `typeof` Type Operator

Get the type of a value:

```typescript
const config = {
  host: 'localhost',
  port: 3000,
  debug: true
};

type Config = typeof config;
// { host: string; port: number; debug: boolean; }
```

Useful for deriving types from runtime values:

```typescript
const statusCodes = {
  OK: 200,
  NOT_FOUND: 404,
  SERVER_ERROR: 500
} as const;

type StatusCode = typeof statusCodes[keyof typeof statusCodes];
// 200 | 404 | 500
```

## Satisfies Operator

Ensure a value matches a type without changing its inferred type:

```typescript
type Color = 'red' | 'green' | 'blue' | { r: number; g: number; b: number };

const palette = {
  primary: 'red',
  secondary: { r: 0, g: 255, b: 0 }
} satisfies Record<string, Color>;

// palette.primary is inferred as 'red' (not string)
// palette.secondary is inferred as { r: number; g: number; b: number }

const color = palette.primary.toUpperCase(); // OK, TypeScript knows it's string
```

Without `satisfies`, you'd need a type annotation that loses precision:

```typescript
const palette: Record<string, Color> = {
  primary: 'red', // Inferred as Color, not 'red'
  secondary: { r: 0, g: 255, b: 0 }
};

const color = palette.primary.toUpperCase(); // Error: Property 'toUpperCase' does not exist on type 'Color'
```

## `never` Type

Represents values that never occur:

```typescript
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

Useful for exhaustiveness checking:

```typescript
type Shape = 'circle' | 'square';

function area(shape: Shape): number {
  switch (shape) {
    case 'circle':
      return Math.PI;
    case 'square':
      return 1;
    default:
      const exhaustive: never = shape;
      throw new Error(`Unhandled shape: ${exhaustive}`);
  }
}
```

If you add a new shape type but forget to handle it, TypeScript errors at the `never` assignment.

## Type Assertions vs Type Guards

**Type assertion** (compile-time only):

```typescript
const value = 'hello' as string | number;
```

**Type guard** (runtime check):

```typescript
if (typeof value === 'string') {
  // TypeScript knows value is string
}
```

Prefer guards. Assertions can lie. Guards can't.

## Branded Types

Simulate nominal typing:

```typescript
type UserId = string & { readonly __brand: unique symbol };
type PostId = string & { readonly __brand: unique symbol };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createPostId(id: string): PostId {
  return id as PostId;
}

function getUser(id: UserId) {
  // Implementation
}

const userId = createUserId('user-123');
const postId = createPostId('post-456');

getUser(userId); // OK
getUser(postId); // Error: Types don't match
```

The `unique symbol` ensures the brands are distinct.

## Practical Patterns

### State Machine Types

```typescript
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: string }
  | { status: 'error'; error: Error };

function render(state: State) {
  switch (state.status) {
    case 'idle':
      return 'Not started';
    case 'loading':
      return 'Loading...';
    case 'success':
      return `Data: ${state.data}`;
    case 'error':
      return `Error: ${state.error.message}`;
  }
}
```

### Builder Pattern with Fluent API

```typescript
class QueryBuilder {
  private conditions: string[] = [];

  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }

  and(condition: string): this {
    return this.where(condition);
  }

  build(): string {
    return this.conditions.join(' AND ');
  }
}

const query = new QueryBuilder()
  .where('age > 18')
  .and('active = true')
  .build();
```

### Deep Partial

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface Config {
  database: {
    host: string;
    port: number;
    credentials: {
      username: string;
      password: string;
    };
  };
}

const updates: DeepPartial<Config> = {
  database: {
    credentials: {
      password: 'newpassword'
    }
  }
};
```

## What You've Learned

- **Union types** with discriminated unions for powerful pattern matching
- **Intersection types** for combining types
- **Mapped types** for transforming types programmatically
- **Conditional types** for type-level logic
- **Template literal types** for string manipulation at the type level
- **Type guards and assertions** for runtime type narrowing
- **`keyof`, `typeof`, and index access** for deriving types
- **`never` for exhaustiveness checking**
- **Branded types** for nominal typing simulation

These advanced types let you encode complex constraints in the type system. The compiler becomes a verification tool for business logic, not just type checking.

Use these features judiciously. Complexity has a cost. But when used well, advanced types catch entire classes of bugs at compile time.

---

**Next:** [Chapter 8: Classes and OOP (Yes, Really)](./08-classes.md)
