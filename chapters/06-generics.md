# Chapter 6: Generics (or: How I Learned to Stop Worrying and Love <T>)

Generics look intimidating. Those angle brackets, the single-letter type names, the academic terminology. But strip away the syntax, and generics are just **parameterized types**—types that take arguments, like functions take arguments.

If you understand functions with parameters, you understand generics.

## The Problem Generics Solve

Without generics, you have two bad options:

**Option 1: Type-specific functions**

```typescript
function identityString(value: string): string {
  return value;
}

function identityNumber(value: number): number {
  return value;
}

function identityBoolean(value: boolean): boolean {
  return value;
}
```

Repetitive. Doesn't scale.

**Option 2: `any`**

```typescript
function identity(value: any): any {
  return value;
}

const result = identity(42); // Type: any
// Lost all type information
```

Works, but you've thrown away the type system.

**Generics: The solution**

```typescript
function identity<T>(value: T): T {
  return value;
}

const num = identity(42);        // Type: number
const str = identity('hello');   // Type: string
const bool = identity(true);     // Type: boolean
```

One function. Full type safety. The `<T>` is a type parameter—a placeholder for whatever type you pass in.

## Generic Functions

The syntax: `<TypeParameter>` before the parameter list.

```typescript
function wrap<T>(value: T): { value: T } {
  return { value };
}

const wrapped = wrap(42);
// Type: { value: number }

console.log(wrapped.value); // 42
```

TypeScript **infers** `T` from the argument. You don't need to specify it explicitly:

```typescript
wrap<number>(42);  // Explicit (unnecessary here)
wrap(42);          // Inferred (better)
```

### Multiple Type Parameters

```typescript
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const p1 = pair(1, 'hello');
// Type: [number, string]

const p2 = pair('foo', true);
// Type: [string, boolean]
```

Convention uses `T`, `U`, `V`, etc. But you can use descriptive names:

```typescript
function pair<First, Second>(first: First, second: Second): [First, Second] {
  return [first, second];
}
```

Single letters are convention, not law.

### Generic Constraints

Sometimes you need to restrict what `T` can be:

```typescript
function getLength<T>(value: T): number {
  return value.length; // Error: Property 'length' does not exist on type 'T'
}
```

`T` could be anything. Not everything has `.length`.

**Solution: Constrain `T`**

```typescript
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength('hello');     // OK (string has .length)
getLength([1, 2, 3]);   // OK (array has .length)
getLength(42);          // Error: Argument of type 'number' is not assignable to parameter of type '{ length: number; }'
```

`T extends { length: number }` means "`T` must have a `length` property that's a number."

You can constrain to specific types:

```typescript
function logValue<T extends string | number>(value: T): void {
  console.log(value);
}

logValue('hello'); // OK
logValue(42);      // OK
logValue(true);    // Error
```

### Using Type Parameters in Constraints

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Alice', age: 30 };

const name = getProperty(user, 'name'); // Type: string
const age = getProperty(user, 'age');   // Type: number

getProperty(user, 'email'); // Error: Argument of type '"email"' is not assignable to parameter of type '"name" | "age"'
```

`K extends keyof T` means "`K` must be a key of `T`."

`T[K]` means "the type of property `K` on `T`."

This is **type-safe property access**. No runtime errors. The compiler prevents invalid keys.

## Generic Interfaces

```typescript
interface Box<T> {
  value: T;
}

const numBox: Box<number> = { value: 42 };
const strBox: Box<string> = { value: 'hello' };
```

You must specify the type parameter when using a generic interface (unlike functions, where it's inferred).

### API Response Example

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

interface User {
  id: string;
  name: string;
}

async function fetchUser(id: string): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return { success: true, data };
}

const result = await fetchUser('123');
if (result.success && result.data) {
  console.log(result.data.name); // TypeScript knows result.data is User
}
```

One `ApiResponse` type. Works with any data type.

## Generic Type Aliases

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return { ok: false, error: 'Division by zero' };
  }
  return { ok: true, value: a / b };
}

const result = divide(10, 2);
if (result.ok) {
  console.log(result.value); // Type: number
} else {
  console.log(result.error); // Type: string
}
```

Notice `E = Error`. That's a **default type parameter**. If you don't specify `E`, it defaults to `Error`.

```typescript
type Result<T> = Result<T, Error>; // Simplified version
```

## Generic Classes

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numStack = new Stack<number>();
numStack.push(1);
numStack.push(2);
console.log(numStack.pop()); // 2

const strStack = new Stack<string>();
strStack.push('hello');
strStack.push('world');
console.log(strStack.pop()); // 'world'
```

One class. Multiple types. Type-safe operations.

## Arrays Are Generic

You've been using generics all along:

```typescript
const numbers: Array<number> = [1, 2, 3];
// Equivalent to: number[]

const strings: Array<string> = ['a', 'b', 'c'];
// Equivalent to: string[]
```

`Array<T>` is a built-in generic. `T[]` is syntactic sugar.

## Promises Are Generic

```typescript
async function fetchData(): Promise<string> {
  const response = await fetch('/api/data');
  return response.text();
}

const data = await fetchData(); // Type: string
```

`Promise<T>` wraps a value of type `T`.

## Generic Utility Types (Revisited)

TypeScript's built-in utility types are all generic:

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Record<K extends string | number | symbol, T> = {
  [P in K]: T;
};
```

These are **mapped types** (Chapter 7). For now, understand they're generic—they take a type and transform it.

## Practical Patterns

### Repository Pattern

```typescript
interface Repository<T> {
  find(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

interface User {
  id: string;
  name: string;
  email: string;
}

class UserRepository implements Repository<User> {
  async find(id: string): Promise<User | null> {
    // Implementation
    return null;
  }

  async findAll(): Promise<User[]> {
    // Implementation
    return [];
  }

  async save(user: User): Promise<User> {
    // Implementation
    return user;
  }

  async delete(id: string): Promise<void> {
    // Implementation
  }
}
```

One interface. Works for any entity type.

### State Management

```typescript
interface State<T> {
  value: T;
  update(newValue: T): void;
  subscribe(listener: (value: T) => void): () => void;
}

function createState<T>(initialValue: T): State<T> {
  let value = initialValue;
  const listeners: Array<(value: T) => void> = [];

  return {
    get value() {
      return value;
    },
    update(newValue: T) {
      value = newValue;
      listeners.forEach(listener => listener(value));
    },
    subscribe(listener: (value: T) => void) {
      listeners.push(listener);
      return () => {
        const index = listeners.indexOf(listener);
        if (index > -1) {
          listeners.splice(index, 1);
        }
      };
    }
  };
}

const counter = createState(0);
counter.subscribe(value => console.log('Counter:', value));
counter.update(1); // Logs: Counter: 1
```

### Option/Maybe Type

```typescript
type Option<T> = Some<T> | None;

interface Some<T> {
  kind: 'some';
  value: T;
}

interface None {
  kind: 'none';
}

function some<T>(value: T): Option<T> {
  return { kind: 'some', value };
}

function none(): Option<never> {
  return { kind: 'none' };
}

function unwrap<T>(option: Option<T>, defaultValue: T): T {
  return option.kind === 'some' ? option.value : defaultValue;
}

const maybeUser = some({ name: 'Alice' });
const user = unwrap(maybeUser, { name: 'Guest' });
console.log(user.name); // Alice
```

## Generic Constraints in Practice

### Ensuring Properties Exist

```typescript
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

const users = [
  { id: '1', name: 'Alice' },
  { id: '2', name: 'Bob' }
];

const user = findById(users, '1');
// Type: { id: string; name: string } | undefined
```

### Constructor Constraints

```typescript
interface Constructable<T> {
  new (...args: any[]): T;
}

function create<T>(ctor: Constructable<T>, ...args: any[]): T {
  return new ctor(...args);
}

class User {
  constructor(public name: string, public age: number) {}
}

const user = create(User, 'Alice', 30);
// Type: User
```

### Extending Built-in Types

```typescript
function first<T extends any[]>(arr: T): T[0] | undefined {
  return arr[0];
}

const nums = [1, 2, 3];
const firstNum = first(nums); // Type: number | undefined

const strs = ['a', 'b', 'c'];
const firstStr = first(strs); // Type: string | undefined
```

## Variance (Advanced but Important)

Generics have **variance**—rules about when one generic type can substitute for another.

### Covariance (Arrays are covariant)

```typescript
class Animal {
  name: string = 'animal';
}

class Dog extends Animal {
  bark() {
    console.log('Woof!');
  }
}

const dogs: Dog[] = [new Dog()];
const animals: Animal[] = dogs; // OK (covariant)

// But this can cause problems:
animals.push(new Animal()); // Allowed, but now dogs array has a non-Dog
dogs[1].bark(); // Runtime error: bark doesn't exist on Animal
```

TypeScript allows this (for compatibility with JavaScript), but it's technically unsound.

### Invariance (Ideally, this should be enforced)

Properly, generic types should be invariant—`T[]` is not assignable to `U[]` unless `T` is exactly `U`.

In practice, TypeScript is pragmatic. Arrays are covariant. Functions are contravariant in parameters, covariant in return types.

**Takeaway:** Be careful when mixing subclasses and generics. TypeScript won't always catch mistakes.

## Generic Default Parameters

```typescript
interface ApiResponse<T = unknown> {
  data: T;
}

const response1: ApiResponse<User> = { data: { id: '1', name: 'Alice' } };
const response2: ApiResponse = { data: 'anything' }; // T defaults to unknown
```

Defaults are useful for library APIs where the user might not always specify a type.

## Conditional Types (Preview)

Generics can have conditional logic:

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

We'll cover this in Chapter 7, but know that generics can be much more powerful than simple placeholders.

## When NOT to Use Generics

### Over-Generalization

```typescript
// Bad: unnecessary generic
function add<T>(a: T, b: T): T {
  return (a as any) + (b as any);
}

// Good: just use the right type
function add(a: number, b: number): number {
  return a + b;
}
```

If you're using `any` inside a generic, you probably don't need the generic.

### Premature Abstraction

Don't add generics "just in case." Add them when you have multiple concrete use cases.

```typescript
// Bad: speculative generic
interface Repository<T, K> {
  find(key: K): Promise<T | null>;
}

// Good: start specific, generalize later
interface UserRepository {
  find(id: string): Promise<User | null>;
}
```

Start concrete. Abstract when patterns emerge.

### Readability

```typescript
// Hard to read
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Sometimes simpler is better
type UpdateUser = {
  name?: string;
  email?: string;
  age?: number;
};
```

Generics add cognitive load. Use them when the benefit (reusability, type safety) outweighs the cost (complexity).

## Common Pitfalls

### Forgetting Constraints

```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

merge('hello', 'world'); // Compiles, but runtime error (strings aren't spreadable)
```

Better:

```typescript
function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

merge('hello', 'world'); // Error: Argument of type 'string' is not assignable to parameter of type 'object'
```

### Overusing `any` in Generic Implementations

```typescript
// Bad: defeats the purpose
function identity<T>(value: T): T {
  const temp: any = value;
  return temp;
}

// Good: just use T
function identity<T>(value: T): T {
  return value;
}
```

If your generic implementation uses `any`, rethink the design.

### Type Parameter Shadowing

```typescript
class Container<T> {
  value: T;

  constructor(value: T) {
    this.value = value;
  }

  // Bad: shadows class T
  map<T>(fn: (value: T) => T): Container<T> {
    return new Container(fn(this.value as any));
  }
}
```

The method's `T` shadows the class's `T`. Use a different name:

```typescript
class Container<T> {
  value: T;

  constructor(value: T) {
    this.value = value;
  }

  map<U>(fn: (value: T) => U): Container<U> {
    return new Container(fn(this.value));
  }
}
```

## What You've Learned

- **Generics are parameterized types**—types that take arguments
- **`<T>` is a type parameter**, like function parameters but for types
- **Constraints** (`T extends U`) restrict what types can be used
- **Type inference** usually figures out generic parameters automatically
- **Generic interfaces, classes, and functions** enable reusable, type-safe code
- **Variance** (covariance, contravariance) affects how generic types relate
- **Don't over-generic**—start specific, generalize when needed

Generics are TypeScript's superpower for abstraction. They let you write code once and reuse it with full type safety across many types.

The syntax is terse. The concepts are simple. The applications are endless.

---

**Next:** [Chapter 7: Advanced Types and the Compiler's Bag of Tricks](./07-advanced-types.md)
