---
layout: chapter
title: "Your TypeScript Toolchain"
chapter_number: 3
permalink: /chapters/toolchain/
---

# Chapter 3: Your TypeScript Toolchain

**Traditional approach:** You don't run TypeScript. You run JavaScript. TypeScript is the step in between—a compiler, a type checker, and a development-time assistant rolled into one.

**Deno approach:** You *do* run TypeScript directly. No build step required.

Both approaches have their place. Understanding the toolchain helps you debug problems, optimize your workflow, and make informed decisions about project setup.

## The Big Picture

### Traditional Flow (Node.js, Browser)

```
.ts files → TypeScript Compiler (tsc) → .js files → Runtime (Node/Browser)
              ↓
         Type checking errors (compile time)
```

The compiler does two jobs:
1. **Type checking** - Analyze your code and report type errors
2. **Transpilation** - Convert TypeScript to JavaScript

These are **separate** operations. You can type-check without generating JS. You can generate JS even if there are type errors (it's a warning, not a blocker, unless you configure it otherwise).

### Modern Flow (Deno, Bun)

```
.ts files → Runtime (Deno/Bun) → Runs TypeScript directly
              ↓
         Type checking (on-demand or in editor)
```

Deno and Bun understand TypeScript natively. No compilation step. Types are checked by your editor or on-demand, but the runtime strips them and executes your code directly.

We'll cover both approaches, starting with the traditional one (which still dominates the ecosystem), then diving into Deno.

## Installing TypeScript

```bash
# Install globally (not recommended for projects)
npm install -g typescript

# Install locally in a project (recommended)
npm install --save-dev typescript

# Check version
npx tsc --version
```

Use local installations. Global installs cause version mismatches across projects. Your team should all use the same TypeScript version.

## The Compiler: `tsc`

The TypeScript compiler is a single command: `tsc`.

```bash
# Compile a single file
tsc hello.ts
# Creates hello.js

# Compile all files in a project (uses tsconfig.json)
tsc

# Watch mode - recompile on file changes
tsc --watch

# Just check types, don't emit JavaScript
tsc --noEmit
```

### Common Flags

```bash
# Specify target JavaScript version
tsc --target ES2020 hello.ts

# Specify module system
tsc --module commonjs hello.ts

# Output to specific directory
tsc --outDir dist hello.ts

# Enable strict mode (recommended)
tsc --strict hello.ts

# Show all compiler options
tsc --help
```

In practice, you rarely use CLI flags. You use `tsconfig.json`.

## The Configuration File: `tsconfig.json`

Every TypeScript project needs a `tsconfig.json`. It tells the compiler what to compile and how.

Create one:

```bash
npx tsc --init
```

This generates a heavily commented `tsconfig.json` with defaults. Here's a minimal, practical version:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Let's break down the critical options.

### `target` - JavaScript version to emit

```json
"target": "ES2022"
```

TypeScript compiles **down** to older JavaScript versions. If you use `async/await` but target ES5, the compiler generates a state machine to emulate it.

Common values (as of 2025):
- `ES5` - Ancient browser support (IE11) - rarely needed now
- `ES2015` / `ES6` - Legacy baseline
- `ES2022` - Modern baseline (top-level await, class fields, private fields)
- `ES2023` - Latest stable features
- `ESNext` - Bleeding edge (avoid in production, moves over time)

**Rule of thumb:** Target the oldest environment you support. For Node.js 18+ (LTS as of 2025), use `ES2022`. For modern browsers (2025), use `ES2022` or `ES2023`.

### `module` - Module system

```json
"module": "commonjs"
```

JavaScript has multiple module systems. TypeScript supports them all:

- `commonjs` - Node.js traditional (`require`/`module.exports`)
- `ES2015` / `ES6` / `ESNext` - ECMAScript modules (`import`/`export`)
- `AMD` - Browser loader (RequireJS, mostly legacy)
- `UMD` - Universal (works everywhere, rare now)
- `node16` / `nodenext` - Node.js native ESM support

**For Node.js projects:** Use `commonjs` unless you're using ESM (package.json has `"type": "module"`).

**For browser projects with bundlers (Webpack, Vite, etc.):** Use `ESNext` and let the bundler handle it.

### `lib` - Available APIs

```json
"lib": ["ES2022", "DOM", "DOM.Iterable"]
```

`lib` tells TypeScript what global APIs exist. If you use `fetch`, `localStorage`, or `document`, you need `"DOM"`. For iterating over DOM collections (like `NodeList`), add `"DOM.Iterable"`. If you use `Promise`, `Map`, or `Array.flatMap`, you need the appropriate ES version.

**For Node.js:**
```json
"lib": ["ES2022"]
```

**For browser:**
```json
"lib": ["ES2022", "DOM", "DOM.Iterable"]
```

### `strict` - All the strictness

```json
"strict": true
```

This enables all strict type checking options:
- `strictNullChecks` - `null` and `undefined` must be handled explicitly
- `strictFunctionTypes` - Function parameter bivariance disabled
- `strictBindCallApply` - `bind`, `call`, `apply` are type-checked
- `strictPropertyInitialization` - Class properties must be initialized
- `noImplicitThis` - `this` must have a known type
- `alwaysStrict` - Emit `"use strict"` in JS
- `noImplicitAny` - Variables can't default to `any`

**Always use `"strict": true`** for new projects. It catches real bugs. Yes, it's more demanding. That's the point.

For migrating JavaScript, you might start with `"strict": false` and enable options incrementally.

### `outDir` and `rootDir` - File organization

```json
"outDir": "./dist",
"rootDir": "./src"
```

- `rootDir` - Where your `.ts` source files live
- `outDir` - Where compiled `.js` files go

This keeps source and build artifacts separate:

```
project/
  src/
    index.ts
    utils.ts
  dist/
    index.js
    utils.js
  tsconfig.json
```

### `esModuleInterop` - CommonJS/ESM compatibility

```json
"esModuleInterop": true
```

JavaScript module systems are messy. CommonJS (`require`) and ES modules (`import`) have subtle incompatibilities. This flag papers over some of them.

Without it:
```typescript
import * as express from 'express'; // Required
```

With it:
```typescript
import express from 'express'; // Cleaner
```

**Just enable it.** It makes life easier.

### `skipLibCheck` - Skip checking `.d.ts` files

```json
"skipLibCheck": true
```

TypeScript checks type definition files (`.d.ts`) from libraries in `node_modules`. This can be slow and cause errors in third-party types you can't fix.

`skipLibCheck` skips them, speeding up compilation and avoiding false positives.

**Enable it unless you're debugging library type issues.**

### `include` and `exclude` - What to compile

```json
"include": ["src/**/*"],
"exclude": ["node_modules", "dist", "**/*.spec.ts"]
```

- `include` - Glob patterns for files to compile
- `exclude` - Patterns to ignore

By default, `tsc` compiles everything. `include`/`exclude` scope it down.

**Common patterns:**
```json
{
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts", "**/*.spec.ts"]
}
```

## Full Example: Node.js Project

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",

    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,

    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    "resolveJsonModule": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,

    "sourceMap": true,
    "declaration": true,
    "declarationMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Bonus options explained:

**`sourceMap: true`** - Generate `.js.map` files for debugging. Your IDE can map compiled JS back to TypeScript source.

**`declaration: true`** - Generate `.d.ts` files. Required if you're publishing a library.

**`declarationMap: true`** - Generate `.d.ts.map` files. Lets IDEs jump from your library's types to its TypeScript source.

**`resolveJsonModule: true`** - Allow `import config from './config.json'`.

**`moduleResolution: "node"`** - Use Node.js module resolution (default for `module: "commonjs"`).

## Full Example: Browser Project (with bundler)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],

    "jsx": "react-jsx",
    "outDir": "./dist",
    "rootDir": "./src",

    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    "moduleResolution": "node",
    "resolveJsonModule": true,
    "allowJs": true,
    "noEmit": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### Differences from Node.js config:

**`"module": "ESNext"`** - Bundler handles modules, output ESNext.

**`"lib": ["ES2020", "DOM", "DOM.Iterable"]`** - Browser APIs available.

**`"jsx": "react-jsx"`** - For React projects (Chapter 12 covers this).

**`"noEmit": true`** - Don't generate JS files (bundler does this). Just type-check.

**`"allowJs": true`** - Allow importing `.js` files alongside `.ts` (useful during migration).

## Editor Integration

TypeScript's killer feature: **real-time type checking in your editor.**

### VS Code (best-in-class support)

TypeScript is built into VS Code. Open a `.ts` file and it just works.

**Useful features:**
- Hover over variables to see types
- Autocomplete with inline documentation
- Jump to definition (F12)
- Find all references (Shift+F12)
- Rename symbol (F2) - renames across files
- Quick fixes (Cmd/Ctrl+.) - auto-import, add missing types

**TypeScript version:** VS Code ships with a TypeScript version, but your project might use a different one. Use the workspace version:

1. Open any `.ts` file
2. Bottom-right corner: click the TypeScript version number
3. Select "Use Workspace Version"

### Other Editors

**WebStorm:** First-class TypeScript support, similar to VS Code.

**Sublime Text:** Install LSP + LSP-typescript package.

**Vim/Neovim:** Use CoC (Conquer of Completion) or native LSP with `typescript-language-server`.

**Emacs:** Use `tide` or `lsp-mode` with `typescript-language-server`.

## The TypeScript Language Server

Your editor doesn't run `tsc`. It runs the **TypeScript Language Server** (`tsserver`), which provides:
- Real-time type checking
- Autocomplete
- Refactoring
- Code navigation

The language server is bundled with TypeScript. When your editor highlights an error or suggests a type, that's the language server at work.

## Build Tools and TypeScript

In real projects, you rarely run `tsc` directly. You integrate it into a build tool.

### With Webpack

```bash
npm install --save-dev typescript ts-loader
```

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: ['.ts', '.js']
  }
};
```

### With Vite

```bash
npm create vite@latest my-app -- --template vanilla-ts
```

Vite has built-in TypeScript support. Just write `.ts` files.

### With esbuild

```bash
npm install --save-dev esbuild
```

```bash
esbuild src/index.ts --bundle --outfile=dist/index.js
```

esbuild compiles TypeScript **but doesn't type-check**. Run `tsc --noEmit` separately for type checking.

### With Node.js (ts-node)

```bash
npm install --save-dev ts-node
```

```bash
npx ts-node src/index.ts
```

`ts-node` runs TypeScript files directly (compiles on-the-fly). Great for development, not for production.

**For production:** Compile with `tsc`, run the JS.

### With Testing Frameworks

**Jest:**
```bash
npm install --save-dev jest ts-jest @types/jest
```

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node'
};
```

**Vitest:**
Built-in TypeScript support. Just works.

## Type Definitions for Libraries

Most npm packages are JavaScript. TypeScript needs type definitions.

### Option 1: Built-in types

Some packages (especially newer ones) include TypeScript types:

```json
// package.json
{
  "types": "dist/index.d.ts"
}
```

No extra installation needed.

### Option 2: DefinitelyTyped (@types)

Most popular packages have community-maintained types on DefinitelyTyped:

```bash
npm install --save-dev @types/node
npm install --save-dev @types/react
npm install --save-dev @types/express
```

The `@types/` package name matches the library name.

### Option 3: No types (manual declarations)

For obscure packages without types, you can declare them yourself:

```typescript
// src/types/my-library.d.ts
declare module 'my-library' {
  export function doSomething(x: string): number;
}
```

Or just use `any`:

```typescript
// src/types/my-library.d.ts
declare module 'my-library';

// Now you can import it (as any)
import lib from 'my-library';
```

## Common Compiler Options You'll Encounter

### `allowJs` and `checkJs`

```json
{
  "allowJs": true,   // Allow importing .js files
  "checkJs": false   // Type-check .js files (off by default)
}
```

Useful during migration. You can mix `.js` and `.ts` files.

### `noEmit`

```json
{
  "noEmit": true  // Don't generate JS files
}
```

Use when a bundler handles compilation. TypeScript just type-checks.

### `incremental`

```json
{
  "incremental": true,  // Cache type info for faster recompilation
  "tsBuildInfoFile": ".tsbuildinfo"
}
```

Speeds up `tsc` by caching. Useful for large projects.

### `paths` - Module aliases

```json
{
  "baseUrl": ".",
  "paths": {
    "@/*": ["src/*"],
    "@components/*": ["src/components/*"]
  }
}
```

Allows:
```typescript
import Button from '@components/Button';
// Instead of
import Button from '../../components/Button';
```

**Caveat:** TypeScript resolves these at compile time, but Node.js/bundlers might not. You may need extra config (e.g., `tsconfig-paths` for Node, or Webpack aliases).

## Debugging TypeScript

### Source Maps

With `"sourceMap": true`, the compiler generates `.js.map` files. Debuggers (Chrome DevTools, VS Code debugger) use these to map compiled JS back to TypeScript source.

```json
{
  "sourceMap": true
}
```

Now when you set breakpoints or inspect errors, you see TypeScript code, not compiled JS.

### VS Code Debugger

`.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "preLaunchTask": "tsc: build - tsconfig.json",
      "program": "${workspaceFolder}/dist/index.js",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "sourceMaps": true
    }
  ]
}
```

Press F5, and you're debugging TypeScript with full source map support.

## Performance Tips

TypeScript can be slow on large codebases. Some tricks:

### 1. Use Project References (monorepos)

For projects with multiple packages, `tsconfig.json` can reference others:

```json
{
  "references": [
    { "path": "../shared" },
    { "path": "../api" }
  ]
}
```

Enables **incremental builds**—unchanged packages aren't recompiled.

### 2. Skip Type Checking Libraries

```json
{
  "skipLibCheck": true
}
```

Huge speed boost, especially with many dependencies.

### 3. Use `tsc --build` for Multi-Project Setups

```bash
tsc --build --watch
```

Watches all referenced projects and recompiles only what changed.

### 4. Increase Memory Limit

For giant codebases:

```bash
NODE_OPTIONS=--max-old-space-size=8192 tsc
```

Gives Node.js more memory for type checking.

## Linting and Formatting

TypeScript catches type errors. Linters catch style issues and potential bugs.

### ESLint with TypeScript

```bash
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended'
  ],
  parserOptions: {
    project: './tsconfig.json'
  }
};
```

### Prettier

```bash
npm install --save-dev prettier
```

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5"
}
```

Prettier formats code. ESLint catches logic issues. Use both.

## Common Errors and What They Mean

### `Cannot find module 'X' or its corresponding type declarations`

**Cause:** Missing type definitions.

**Fix:**
```bash
npm install --save-dev @types/X
```

Or declare the module:
```typescript
declare module 'X';
```

### `Object is possibly 'null'` or `'undefined'`

**Cause:** Strict null checking.

**Fix:** Check before using:
```typescript
if (value) {
  value.doSomething();
}
```

Or use optional chaining:
```typescript
value?.doSomething();
```

### `Property 'X' does not exist on type 'Y'`

**Cause:** Object doesn't have that property (or TypeScript thinks it doesn't).

**Fix:** Add the property to the type, or use a type assertion if you know better:
```typescript
(obj as any).X
```

### `Type 'X' is not assignable to type 'Y'`

**Cause:** You're trying to assign a value to a variable with an incompatible type.

**Fix:** Either change the type or convert the value.

## Deno: TypeScript Without the Build Step

Deno is a modern JavaScript/TypeScript runtime built by Ryan Dahl (Node.js creator) that understands TypeScript natively. No `tsc`, no build step, no configuration required.

### Why Deno Matters

Deno eliminates the entire compilation phase. Write `.ts` files, run them directly:

```bash
# Install Deno
curl -fsSL https://deno.land/install.sh | sh

# Run TypeScript directly
deno run main.ts

# That's it. No tsconfig.json, no tsc, no build step.
```

### How It Works

Deno internally:
1. Strips TypeScript type annotations
2. Caches the transformed JavaScript
3. Executes it

Type checking happens **separately** (in your editor or on-demand):

```bash
# Just run (type errors are warnings, not blockers)
deno run main.ts

# Type check explicitly
deno check main.ts

# Type check and run
deno run --check main.ts
```

This is **fundamentally different** from Node.js. Deno prioritizes fast execution—type checking is optional.

### Deno Project Structure

No `package.json`. No `node_modules`. No `tsconfig.json` (usually).

```typescript
// main.ts
import { serve } from "https://deno.land/std@0.208.0/http/server.ts";

serve((req: Request) => {
  return new Response("Hello from Deno!");
});
```

```bash
deno run --allow-net main.ts
```

Dependencies are URLs. Deno downloads and caches them. Type definitions are included (or fetched from a CDN).

### Permissions

Deno is secure by default. You must explicitly grant permissions:

```bash
# Network access
deno run --allow-net server.ts

# File system read
deno run --allow-read reader.ts

# File system write
deno run --allow-write writer.ts

# Environment variables
deno run --allow-env script.ts

# All permissions (like Node.js)
deno run -A script.ts
```

This makes Deno safer for running untrusted code. TypeScript's type system doesn't provide security—Deno's permissions do.

### Configuring Deno (when needed)

For advanced projects, you can use `deno.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "lib": ["deno.window", "dom"],
    "jsx": "react-jsx"
  },
  "imports": {
    "std/": "https://deno.land/std@0.208.0/",
    "react": "https://esm.sh/react@18.2.0"
  },
  "tasks": {
    "dev": "deno run --watch --allow-net server.ts",
    "test": "deno test --allow-read"
  }
}
```

But for most projects, you don't need this. Deno's defaults are sensible.

### Type Definitions in Deno

Deno has built-in types for:
- Web APIs (fetch, Request, Response, etc.)
- Deno-specific APIs (Deno.readFile, Deno.serve, etc.)

For npm packages (Deno supports them via `npm:` specifier):

```typescript
import express from "npm:express@4.18.2";

const app = express();
app.get("/", (req, res) => res.send("Hello"));
app.listen(3000);
```

Deno automatically fetches types from `@types/*` if available.

For CDN packages (esm.sh, unpkg, etc.), types are usually bundled or fetched automatically:

```typescript
import React from "https://esm.sh/react@18.2.0";
// Types included automatically
```

### Deno vs Node.js: When to Use Which

**Use Deno when:**
- Starting a new TypeScript project
- You want zero configuration
- You value security (permissions model)
- You're building CLI tools or scripts
- You want modern defaults (ESM, top-level await, Web APIs)

**Use Node.js when:**
- You have an existing codebase
- You need the npm ecosystem's full depth (some packages don't work in Deno)
- You're integrating with Node-specific tools (certain bundlers, frameworks)
- Your team already knows Node.js tooling

**The reality:** Node.js dominates production. Deno is growing. Knowing both is valuable.

### Deno and TypeScript Features

Deno uses the latest TypeScript compiler internally, so **all TypeScript features work**:

```typescript
// Generics
function identity<T>(value: T): T {
  return value;
}

// Union types
type Status = "pending" | "success" | "error";

// Async/await (top-level in Deno)
const response = await fetch("https://api.example.com/data");
const data = await response.json();

// Decorators (experimental)
function logged(target: any, key: string) {
  console.log(`${key} was called`);
}

class Example {
  @logged
  method() {}
}
```

### Deno Testing (Built-in)

Deno includes a test runner. No Jest, no Vitest, no configuration:

```typescript
// math_test.ts
import { assertEquals } from "https://deno.land/std@0.208.0/assert/mod.ts";

Deno.test("addition works", () => {
  assertEquals(2 + 2, 4);
});

Deno.test("async test", async () => {
  const result = await Promise.resolve(42);
  assertEquals(result, 42);
});
```

```bash
deno test
```

Types work automatically in tests. No `@types/jest` needed.

### Deno Formatter and Linter (Built-in)

```bash
# Format code (like Prettier)
deno fmt

# Lint code (like ESLint)
deno lint

# Check types
deno check main.ts
```

All built-in. No configuration files. Opinionated defaults.

### VSCode with Deno

Install the official Deno extension:

```json
// .vscode/settings.json
{
  "deno.enable": true,
  "deno.lint": true,
  "deno.unstable": false
}
```

Now VS Code understands Deno's URL imports, permissions, and APIs.

### Deno Deploy (Bonus: Serverless TypeScript)

Deno has a serverless platform (Deno Deploy) that runs TypeScript **directly in production**:

```typescript
// server.ts
Deno.serve((req: Request) => {
  return new Response("Hello from the edge!");
});
```

Deploy:
```bash
deployctl deploy --project=my-project server.ts
```

No build step. No Docker. TypeScript runs at the edge. This is where Deno truly shines—**production TypeScript without compilation**.

### Migrating from Node.js to Deno

It's usually not worth migrating existing projects. But for **new** projects:

**Node.js:**
```bash
npm init -y
npm install typescript @types/node --save-dev
npx tsc --init
# Configure tsconfig.json
# Configure build scripts
# Configure test framework
npm install express
npm install --save-dev @types/express
```

**Deno:**
```bash
# That's it. Start writing .ts files.
```

The development velocity difference is significant.

### Bun: The Third Option

Bun (another modern runtime) also runs TypeScript directly:

```bash
bun run server.ts
```

Bun is **extremely fast** (uses JavaScriptCore instead of V8) and has built-in bundler, test runner, and package manager. It's npm-compatible (unlike Deno's URL imports).

**Bun vs Deno:**
- Bun: Faster, npm-compatible, less mature
- Deno: More mature, security-focused, standards-based

Both eliminate the TypeScript build step. Both are worth watching.

## Choosing Your Toolchain

**For new TypeScript projects:**
- **Deno:** Zero config, maximum simplicity
- **Bun:** Maximum speed, npm compatibility
- **Node.js + Vite/esbuild:** Battle-tested, ecosystem depth

**For existing JavaScript projects:**
- **Node.js + TypeScript:** Standard migration path
- **Gradual adoption:** Start with `allowJs`, add types incrementally

**For frontend projects:**
- **Vite:** Modern bundler, instant TypeScript support
- **Next.js/Remix:** Framework handles TypeScript automatically

**For libraries:**
- **Node.js + tsc:** Generate .d.ts files for consumers
- **tsup/unbuild:** Modern bundling with types

The "best" toolchain depends on your project. Deno is simplest. Node.js is most supported. Bun is fastest.

## What You've Learned (Expanded)

- **Traditional flow:** `tsc` compiles `.ts` → `.js`, runtime executes JS
- **Deno/Bun flow:** Runtime executes `.ts` directly, no build step
- **`tsconfig.json`** configures the compiler (Node.js approach)
- **`deno.json`** configures Deno (optional, rarely needed)
- **`strict: true`** applies to both approaches
- **Editor integration** works with all toolchains (VS Code is best)
- **Type definitions:** `@types/*` for Node, built-in for Deno
- **Deno advantages:** No config, no build, secure by default, modern APIs
- **Node.js advantages:** Mature ecosystem, massive package library, production-proven
- **Choose based on project needs, not hype**

The TypeScript toolchain is evolving. Traditional compilation (tsc) still dominates, but native runtimes (Deno, Bun) are gaining traction. Understanding both prepares you for the ecosystem's future.

---

**Next:** [Chapter 4: Functions: Where Things Get Interesting](./04-functions.md)
