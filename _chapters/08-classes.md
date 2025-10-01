---
layout: chapter
title: "Classes and OOP (Yes, Really)"
chapter_number: 8
permalink: /chapters/classes/
---

# Chapter 8: Classes and OOP (Yes, Really)

Classes in JavaScript are controversial. Are they a betrayal of prototypal inheritance? Syntactic sugar that confuses beginners? Or a pragmatic tool for organizing code?

TypeScript doesn't care about your philosophical stance. It takes JavaScript's class syntax and adds the type safety and features you'd expect from a language with proper OOP support.

Whether you love classes or avoid them, understanding TypeScript's class features matters—because libraries use them, frameworks expect them, and sometimes they're genuinely the right tool.

## Basic Class Syntax

JavaScript and TypeScript classes look similar:

```typescript
class User {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hello, I'm ${this.name}`;
  }
}

const user = new User('Alice', 30);
console.log(user.greet()); // "Hello, I'm Alice"
```

Differences from JavaScript:
- Property types are declared (`name: string`)
- Method return types can be annotated (`: string`)
- TypeScript enforces that properties are initialized

## Access Modifiers

TypeScript adds visibility control:

```typescript
class User {
  public name: string;    // Accessible everywhere (default)
  private password: string; // Only accessible inside this class
  protected email: string;  // Accessible in this class and subclasses

  constructor(name: string, password: string, email: string) {
    this.name = name;
    this.password = password;
    this.email = email;
  }

  authenticate(pwd: string): boolean {
    return this.password === pwd; // OK, we're inside the class
  }
}

const user = new User('Alice', 'secret', 'alice@example.com');
console.log(user.name);     // OK (public)
console.log(user.password); // Error: Property 'password' is private
console.log(user.email);    // Error: Property 'email' is protected
```

**Important:** These are **compile-time only**. At runtime, all properties are accessible:

```javascript
// Compiled JS (simplified)
class User {
  constructor(name, password, email) {
    this.name = name;
    this.password = password; // Still here!
    this.email = email;
  }
}
```

For true privacy, use JavaScript private fields (`#`):

```typescript
class User {
  #password: string; // Real privacy (runtime)

  constructor(name: string, password: string) {
    this.#password = password;
  }

  authenticate(pwd: string): boolean {
    return this.#password === pwd;
  }
}

const user = new User('Alice', 'secret');
console.log(user.#password); // Syntax error in JavaScript
```

TypeScript supports `#` syntax. It's true privacy.

## Parameter Properties

Shorthand for declaring and initializing properties:

```typescript
// Verbose
class User {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

// Concise (parameter properties)
class User {
  constructor(
    public name: string,
    public age: number
  ) {}
}
```

The `public` keyword automatically creates and assigns the property. Works with `private`, `protected`, and `readonly` too:

```typescript
class User {
  constructor(
    public name: string,
    private password: string,
    protected email: string,
    readonly id: string
  ) {}
}
```

## Readonly Properties

```typescript
class User {
  readonly id: string;
  name: string;

  constructor(id: string, name: string) {
    this.id = id;
    this.name = name;
  }

  updateName(newName: string) {
    this.name = newName; // OK
    this.id = 'new-id';  // Error: Cannot assign to 'id' because it is a read-only property
  }
}
```

`readonly` prevents modification after initialization. Compile-time only.

## Getters and Setters

```typescript
class User {
  private _age: number;

  constructor(age: number) {
    this._age = age;
  }

  get age(): number {
    return this._age;
  }

  set age(value: number) {
    if (value < 0) {
      throw new Error('Age cannot be negative');
    }
    this._age = value;
  }
}

const user = new User(30);
console.log(user.age); // 30 (calls getter)
user.age = 31;         // Calls setter
user.age = -5;         // Throws error
```

Getters and setters are standard JavaScript. TypeScript just types them.

## Static Members

Belong to the class, not instances:

```typescript
class MathUtils {
  static PI: number = 3.14159;

  static square(x: number): number {
    return x * x;
  }
}

console.log(MathUtils.PI);      // 3.14159
console.log(MathUtils.square(5)); // 25

const utils = new MathUtils();
console.log(utils.PI); // Error: Property 'PI' does not exist on type 'MathUtils'
```

Useful for utility functions and constants.

## Inheritance

Classes can extend other classes:

```typescript
class Animal {
  constructor(public name: string) {}

  move(distance: number): void {
    console.log(`${this.name} moved ${distance}m`);
  }
}

class Dog extends Animal {
  bark(): void {
    console.log('Woof!');
  }
}

const dog = new Dog('Buddy');
dog.move(10); // Inherited from Animal
dog.bark();   // Defined in Dog
```

### Overriding Methods

```typescript
class Animal {
  move(distance: number): void {
    console.log(`Moved ${distance}m`);
  }
}

class Snake extends Animal {
  move(distance: number): void {
    console.log('Slithering...');
    super.move(distance); // Call parent method
  }
}

const snake = new Snake();
snake.move(5);
// "Slithering..."
// "Moved 5m"
```

`super` calls the parent class method.

### Protected Members in Inheritance

```typescript
class Person {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }
}

class Employee extends Person {
  private department: string;

  constructor(name: string, department: string) {
    super(name);
    this.department = department;
  }

  introduce(): string {
    return `I'm ${this.name} from ${this.department}`;
    // Can access protected name here
  }
}

const employee = new Employee('Alice', 'Engineering');
console.log(employee.introduce()); // OK
console.log(employee.name);        // Error: Property 'name' is protected
```

## Abstract Classes

Cannot be instantiated directly. Serve as base classes:

```typescript
abstract class Shape {
  abstract area(): number; // Must be implemented by subclasses

  describe(): string {
    return `Area: ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) {
    super();
  }

  area(): number {
    return this.width * this.height;
  }
}

const circle = new Circle(5);
console.log(circle.describe()); // "Area: 78.53981633974483"

const shape = new Shape(); // Error: Cannot create an instance of an abstract class
```

Abstract classes are like interfaces but can have implementation.

## Implementing Interfaces

Classes can implement interfaces:

```typescript
interface Movable {
  speed: number;
  move(): void;
}

class Car implements Movable {
  speed: number;

  constructor(speed: number) {
    this.speed = speed;
  }

  move(): void {
    console.log(`Driving at ${this.speed} km/h`);
  }
}

class Bicycle implements Movable {
  speed: number;

  constructor(speed: number) {
    this.speed = speed;
  }

  move(): void {
    console.log(`Cycling at ${this.speed} km/h`);
  }
}
```

Multiple interfaces:

```typescript
interface Named {
  name: string;
}

interface Aged {
  age: number;
}

class Person implements Named, Aged {
  constructor(
    public name: string,
    public age: number
  ) {}
}
```

## Class Expressions

Classes can be anonymous:

```typescript
const UserClass = class {
  constructor(public name: string) {}
};

const user = new UserClass('Alice');
```

Rarely used. Named classes are clearer.

## `this` Types

`this` as a return type for chaining:

```typescript
class Calculator {
  private value: number = 0;

  add(x: number): this {
    this.value += x;
    return this;
  }

  subtract(x: number): this {
    this.value -= x;
    return this;
  }

  getResult(): number {
    return this.value;
  }
}

const result = new Calculator()
  .add(10)
  .subtract(3)
  .add(5)
  .getResult();

console.log(result); // 12
```

`this` ensures subclasses maintain chainability:

```typescript
class ScientificCalculator extends Calculator {
  square(): this {
    // Implementation
    return this;
  }
}

const result = new ScientificCalculator()
  .add(5)
  .square()
  .subtract(10)
  .getResult();
```

## Class Type Guards

Check if an object is an instance of a class:

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

## Generic Classes

```typescript
class Box<T> {
  private value: T;

  constructor(value: T) {
    this.value = value;
  }

  getValue(): T {
    return this.value;
  }

  setValue(value: T): void {
    this.value = value;
  }
}

const numBox = new Box<number>(42);
console.log(numBox.getValue()); // 42

const strBox = new Box<string>('hello');
console.log(strBox.getValue()); // 'hello'
```

## Decorators

Decorators are a Stage 3 JavaScript feature. TypeScript 5.0+ supports them natively:

```json
{
  "compilerOptions": {
    // TypeScript 5.0+ uses Stage 3 decorators by default
    // For legacy (experimental) decorators, use:
    "experimentalDecorators": true
  }
}
```

### Class Decorator (Legacy)

```typescript
function logged(constructor: Function) {
  console.log(`Class ${constructor.name} was defined`);
}

@logged
class User {
  constructor(public name: string) {}
}
```

### Method Decorator (Legacy)

```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    return originalMethod.apply(this, args);
  };
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Logs: "Calling add with args: [2, 3]"
// Returns: 5
```

**Note:** Stage 3 decorators (TypeScript 5.0+) have different syntax and capabilities. The examples above show legacy decorators (still widely used). For new projects, consider learning Stage 3 decorator syntax, which includes `accessor` keyword and different decorator signatures.

## When to Use Classes

**Use classes when:**
- You need inheritance (polymorphism)
- You're working with OOP-heavy libraries (Angular, NestJS)
- You want private state with methods
- You're modeling real-world entities with behavior

**Avoid classes when:**
- Simple functions suffice
- You don't need state or inheritance
- Composition over inheritance makes sense
- You prefer functional programming

JavaScript's functional patterns (closures, higher-order functions, modules) often replace classes. Classes aren't mandatory.

## Classes vs Interfaces

**Interfaces** describe shapes. They disappear at compile time.

**Classes** are blueprints AND runtime values. They can be instantiated.

```typescript
interface User {
  name: string;
  greet(): string;
}

// You can't do this:
const user = new User(); // Error: 'User' only refers to a type

// But with a class:
class User {
  constructor(public name: string) {}
  greet(): string {
    return `Hello, ${this.name}`;
  }
}

const user = new User('Alice'); // OK
```

Classes have both **type** and **value** presence.

## Practical Patterns

### Singleton

```typescript
class Database {
  private static instance: Database;

  private constructor() {
    // Private constructor prevents direct instantiation
  }

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  query(sql: string): void {
    console.log(`Executing: ${sql}`);
  }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();

console.log(db1 === db2); // true (same instance)
```

### Factory Pattern

```typescript
abstract class Animal {
  abstract makeSound(): string;
}

class Dog extends Animal {
  makeSound(): string {
    return 'Woof!';
  }
}

class Cat extends Animal {
  makeSound(): string {
    return 'Meow!';
  }
}

class AnimalFactory {
  static createAnimal(type: 'dog' | 'cat'): Animal {
    switch (type) {
      case 'dog':
        return new Dog();
      case 'cat':
        return new Cat();
      default:
        throw new Error('Unknown animal type');
    }
  }
}

const dog = AnimalFactory.createAnimal('dog');
console.log(dog.makeSound()); // "Woof!"
```

### Builder Pattern

```typescript
class HttpRequest {
  private url: string = '';
  private method: string = 'GET';
  private headers: Record<string, string> = {};
  private body?: string;

  setUrl(url: string): this {
    this.url = url;
    return this;
  }

  setMethod(method: string): this {
    this.method = method;
    return this;
  }

  setHeader(key: string, value: string): this {
    this.headers[key] = value;
    return this;
  }

  setBody(body: string): this {
    this.body = body;
    return this;
  }

  async send(): Promise<Response> {
    return fetch(this.url, {
      method: this.method,
      headers: this.headers,
      body: this.body
    });
  }
}

const response = await new HttpRequest()
  .setUrl('/api/users')
  .setMethod('POST')
  .setHeader('Content-Type', 'application/json')
  .setBody(JSON.stringify({ name: 'Alice' }))
  .send();
```

## What You've Learned

- **Classes** in TypeScript are JavaScript classes with types
- **Access modifiers** (`public`, `private`, `protected`) enforce encapsulation (compile-time)
- **Parameter properties** are shorthand for declaring and assigning properties
- **`readonly`** prevents modification after initialization
- **Static members** belong to the class, not instances
- **Inheritance** with `extends` and `super`
- **Abstract classes** can't be instantiated, force implementation
- **Interfaces** can be implemented by classes
- **Generic classes** parameterize over types
- **Decorators** are experimental but powerful

Classes are optional. TypeScript supports them well, but doesn't force you to use them. Choose the paradigm that fits your problem.

---

**Next:** [Chapter 9: Modules, Namespaces, and Declaration Files](./09-modules.md)
