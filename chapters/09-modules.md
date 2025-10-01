# Chapter 9: Modules, Namespaces, and Declaration Files

JavaScript's module story is messy. CommonJS for Node. ES Modules for browsers. AMD for RequireJS. UMD for "universal" compatibility. SystemJS. The list goes on.

TypeScript supports them all. It also adds its own features (namespaces) and a critical tool for library authors: **declaration files** (`.d.ts`).

This chapter covers how TypeScript organizes code across files and how to consume (or create) typed libraries.

## ES Modules (The Modern Standard)

If you're starting fresh, use ES modules:

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;
```

```typescript
// main.ts
import { add, subtract, PI } from './math';

console.log(add(2, 3));    // 5
console.log(subtract(5, 2)); // 3
console.log(PI);            // 3.14159
```

### Default Exports

```typescript
// user.ts
export default class User {
  constructor(public name: string) {}
}
```

```typescript
// main.ts
import User from './user';

const user = new User('Alice');
```

**Prefer named exports.** Default exports are harder to refactor and autocomplete.

### Re-exports

```typescript
// shapes/circle.ts
export class Circle {}

// shapes/rectangle.ts
export class Rectangle {}

// shapes/index.ts
export { Circle } from './circle';
export { Rectangle } from './rectangle';

// Or shorthand
export * from './circle';
export * from './rectangle';
```

```typescript
// main.ts
import { Circle, Rectangle } from './shapes';
```

Barrel exports (index files) organize APIs.

### Import Types

Sometimes you only need a type, not the value:

```typescript
// user.ts
export class User {
  name: string;
}

// main.ts
import type { User } from './user';

const user: User = { name: 'Alice' }; // OK, only using the type

const instance = new User(); // Error: 'User' cannot be used as a value because it was imported using 'import type'
```

`import type` imports only the type. The value is not available. This helps with tree-shaking—unused imports are stripped.

### Inline Type Imports

```typescript
import { type User, fetchUser } from './user';

// User is a type, fetchUser is a value
```

## CommonJS (Node.js Traditional)

TypeScript compiles to CommonJS when `"module": "commonjs"` in `tsconfig.json`.

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}
```

Compiles to:

```javascript
// math.js
exports.add = function(a, b) {
  return a + b;
};
```

Importing:

```typescript
// main.ts
import { add } from './math';
```

Compiles to:

```javascript
// main.js
const { add } = require('./math');
```

TypeScript handles the conversion seamlessly.

### Default Export in CommonJS

```typescript
// user.ts
export default class User {
  constructor(public name: string) {}
}
```

Compiles to:

```javascript
// user.js
class User {
  constructor(name) {
    this.name = name;
  }
}
module.exports = User;
```

With `esModuleInterop: true`, this works:

```typescript
import User from './user';
```

Without it, you need:

```typescript
import * as User from './user';
```

Always enable `esModuleInterop`.

## Module Resolution

TypeScript has multiple strategies for finding modules:

### Classic (Legacy, Avoid)

Rare. Don't use.

### Node (Standard for Node.js)

Mimics Node's resolution:

1. Relative imports (`./`, `../`) resolve from the current file
2. Non-relative imports (`lodash`, `react`) resolve from `node_modules`

```typescript
import { debounce } from 'lodash'; // Looks in node_modules/lodash
import { User } from './models/user'; // Relative path
```

TypeScript looks for:
- `lodash.ts`
- `lodash.tsx`
- `lodash.d.ts`
- `lodash/package.json` (check `types` field)
- `lodash/index.ts`

### Node16/NodeNext (Modern Node.js)

For Node.js with native ESM support:

```json
{
  "compilerOptions": {
    "module": "node16",
    "moduleResolution": "node16"
  }
}
```

Requires explicit file extensions:

```typescript
import { add } from './math.js'; // Must include .js extension
```

Even though the source is `.ts`, imports reference `.js` (what will exist after compilation).

### Bundler (Modern Bundlers)

For Webpack, Vite, etc.:

```json
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "bundler"
  }
}
```

Bundlers handle resolution. TypeScript just type-checks.

## Path Aliases

Clean up import paths:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    }
  }
}
```

```typescript
// Instead of
import { Button } from '../../../components/Button';

// You can write
import { Button } from '@components/Button';
```

**Caveat:** TypeScript resolves these at compile time, but runtimes (Node.js, browsers) don't understand them. You may need extra tooling:
- **Node.js:** `tsconfig-paths` or `module-alias`
- **Webpack/Vite:** Alias configuration
- **Deno:** Import map in `deno.json`

## Namespaces (Legacy)

Before ES modules, TypeScript had **namespaces** (originally called "internal modules"):

```typescript
namespace MathUtils {
  export function add(a: number, b: number): number {
    return a + b;
  }

  export function subtract(a: number, b: number): number {
    return a - b;
  }
}

console.log(MathUtils.add(2, 3)); // 5
```

Namespaces compile to IIFEs (Immediately Invoked Function Expressions):

```javascript
var MathUtils;
(function (MathUtils) {
  function add(a, b) {
    return a + b;
  }
  MathUtils.add = add;
})(MathUtils || (MathUtils = {}));
```

**Don't use namespaces for new code.** Use ES modules. Namespaces exist for legacy compatibility (older libraries, ambient declarations).

### When You Might See Namespaces

Type definitions for global libraries:

```typescript
// @types/jquery/index.d.ts
declare namespace $ {
  function ajax(settings: any): any;
}
```

Augmenting global scope:

```typescript
declare global {
  interface Window {
    myGlobal: string;
  }
}

window.myGlobal = 'value';
```

## Declaration Files (`.d.ts`)

Declaration files describe types without implementation. They're like header files in C/C++.

### Why They Exist

JavaScript libraries have no types. Declaration files add them:

```javascript
// lodash.js (JavaScript)
export function debounce(fn, delay) {
  // Implementation
}
```

```typescript
// lodash.d.ts (TypeScript declaration)
export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void;
```

Now TypeScript knows `debounce`'s signature.

### Creating Declaration Files

For your own libraries:

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}
```

With `"declaration": true` in `tsconfig.json`, TypeScript generates:

```typescript
// math.d.ts
export declare function add(a: number, b: number): number;
```

The `.d.ts` file has types but no implementation. Consumers get types; the `.js` file has runtime code.

### Ambient Declarations

Describe global variables or modules without implementation:

```typescript
// global.d.ts
declare const VERSION: string;
declare function log(message: string): void;
```

Now you can use `VERSION` and `log` without errors:

```typescript
console.log(VERSION); // OK
log('Hello');         // OK
```

This is how you type global scripts or external libraries loaded via `<script>` tags.

### Module Augmentation

Extend existing module types:

```typescript
// express.d.ts
import 'express';

declare module 'express' {
  interface Request {
    user?: { id: string; name: string };
  }
}
```

Now `req.user` is typed in Express middleware:

```typescript
app.get('/profile', (req, res) => {
  if (req.user) {
    res.send(`Hello, ${req.user.name}`);
  }
});
```

### Global Augmentation

```typescript
// globals.d.ts
export {}; // Make this a module

declare global {
  interface Window {
    analytics: {
      track(event: string): void;
    };
  }
}
```

Now `window.analytics` is typed:

```typescript
window.analytics.track('page_view');
```

## DefinitelyTyped (`@types/*`)

The community-maintained repository for type definitions:

```bash
npm install --save-dev @types/node
npm install --save-dev @types/react
npm install --save-dev @types/express
```

Over 8,000 packages have types on DefinitelyTyped. If a library lacks built-in types, check for `@types/[package-name]`.

### How It Works

When you install `@types/lodash`, TypeScript automatically finds it:

```typescript
import { debounce } from 'lodash';
// TypeScript looks for:
// 1. node_modules/lodash/index.d.ts
// 2. node_modules/@types/lodash/index.d.ts
```

### Publishing Your Own Types

If you maintain a library:

**Option 1: Bundle types with your package**

```json
// package.json
{
  "name": "my-library",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts"
}
```

Set `"declaration": true` in `tsconfig.json`. TypeScript generates `.d.ts` files alongside `.js` files.

**Option 2: Publish to DefinitelyTyped**

If you don't control the library, contribute types to DefinitelyTyped:

```bash
git clone https://github.com/DefinitelyTyped/DefinitelyTyped
cd DefinitelyTyped
mkdir types/my-library
cd types/my-library
# Write index.d.ts
# Submit PR
```

## Triple-Slash Directives

Special comments for compiler instructions:

```typescript
/// <reference path="./global.d.ts" />
/// <reference types="node" />
```

These tell TypeScript to include files or types.

**Modern usage is rare.** ES modules and `tsconfig.json` handle most cases.

### When You See Them

- Legacy code
- Global type definitions
- Libraries with complex type dependencies

## Export Assignment (CommonJS Compatibility)

Some CommonJS modules export a single value:

```javascript
// math.js
module.exports = function add(a, b) {
  return a + b;
};
```

Type it with `export =`:

```typescript
// math.d.ts
declare function add(a: number, b: number): number;
export = add;
```

Import with:

```typescript
import add = require('./math');
```

**Avoid this pattern.** Use ES modules.

## Practical Patterns

### Barrel Exports

```typescript
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
```

```typescript
// Consumers import from one place
import { Button, Input, Modal } from './components';
```

Cleans up imports. But adds a hop—every import goes through `index.ts`. Use judiciously.

### Conditional Exports (package.json)

For library authors:

```json
{
  "name": "my-library",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs",
      "types": "./dist/index.d.ts"
    },
    "./utils": {
      "import": "./dist/utils.mjs",
      "types": "./dist/utils.d.ts"
    }
  }
}
```

Different files for ESM vs CommonJS. TypeScript understands this.

### Type-Only Files

```typescript
// types.ts
export interface User {
  id: string;
  name: string;
}

export type Status = 'active' | 'inactive';
```

Centralizes types. No runtime code. Tree-shakes away if unused.

## Common Issues

### Cannot Find Module

```
Error: Cannot find module './math' or its corresponding type declarations.
```

**Causes:**
- File doesn't exist
- Missing file extension (Node16/NodeNext)
- Missing `@types/` package
- Misconfigured `paths` in `tsconfig.json`

**Fix:**
- Check file exists
- Add `.js` extension for Node16/NodeNext
- Install `@types/[package]`
- Verify `baseUrl` and `paths` in `tsconfig.json`

### Circular Dependencies

```typescript
// a.ts
import { B } from './b';
export class A {}

// b.ts
import { A } from './a';
export class B {}
```

TypeScript (and JavaScript) allow this, but it can cause runtime errors. Refactor to break the cycle.

### Default Export Confusion

```typescript
// user.ts
export default class User {}

// main.ts
import { User } from './user'; // Error: Module has no exported member 'User'
```

Default exports aren't named exports. Use:

```typescript
import User from './user';
```

Or:

```typescript
import * as UserModule from './user';
const User = UserModule.default;
```

Better: use named exports.

## What You've Learned

- **ES modules** are the modern standard (`import`/`export`)
- **CommonJS** is Node.js traditional (`require`/`module.exports`)
- **Module resolution** determines how imports are found
- **Path aliases** clean up imports (`@/` instead of `../../`)
- **Namespaces** are legacy; use modules
- **Declaration files** (`.d.ts`) describe types without implementation
- **Ambient declarations** type global variables and scripts
- **Module augmentation** extends existing types
- **DefinitelyTyped** provides types for JavaScript libraries
- **Triple-slash directives** are mostly legacy

TypeScript's module system bridges JavaScript's fractured ecosystem. It supports all module formats, adds type safety, and provides tools for library authors.

---

**Next:** [Chapter 10: The TypeScript Ecosystem](./10-ecosystem.md)
