---
layout: chapter
title: "The TypeScript Ecosystem"
chapter_number: 10
permalink: /chapters/ecosystem/
---

# Chapter 10: The TypeScript Ecosystem

TypeScript isn't just a compiler. It's a thriving ecosystem of tools, libraries, frameworks, and patterns that have evolved around it.

This chapter surveys the landscape: what works well with TypeScript, what requires extra configuration, and where the community is heading.

## Frameworks and Libraries

### React

TypeScript and React are a natural pair. React's component model maps cleanly to TypeScript's type system.

```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
}

function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}
```

**Key packages:**
- `@types/react` - React type definitions
- `@types/react-dom` - ReactDOM type definitions

**Setup:**
```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "lib": ["ES2020", "DOM"]
  }
}
```

More in Chapter 12.

### Vue

Vue 3 is written in TypeScript and has first-class support:

```typescript
import { defineComponent } from 'vue';

export default defineComponent({
  props: {
    message: {
      type: String,
      required: true
    }
  },
  setup(props) {
    return {
      uppercased: props.message.toUpperCase()
    };
  }
});
```

**Setup:**
```bash
npm create vue@latest # Choose TypeScript
```

### Angular

Angular is TypeScript-first. It was designed with TypeScript from the start:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `<h1>{{ title }}</h1>`
})
export class AppComponent {
  title: string = 'Hello Angular';
}
```

No extra setup needed. Angular CLI handles everything.

### Svelte

Svelte has TypeScript support via `svelte-check`:

```svelte
<script lang="ts">
  export let name: string;
  let count: number = 0;

  function increment() {
    count += 1;
  }
</script>

<button on:click={increment}>
  Clicked {count} times by {name}
</button>
```

**Setup:**
```bash
npm create vite@latest my-app -- --template svelte-ts
```

### Node.js

Node.js with TypeScript requires configuration:

```bash
npm install --save-dev typescript @types/node ts-node
npx tsc --init
```

Or use Deno/Bun for zero-config TypeScript.

**Key packages:**
- `@types/node` - Node.js type definitions
- `ts-node` - Run TypeScript directly in Node
- `tsx` - Faster alternative to ts-node

### Express

```bash
npm install express
npm install --save-dev @types/express
```

```typescript
import express, { Request, Response } from 'express';

const app = express();

app.get('/users/:id', (req: Request, res: Response) => {
  const userId = req.params.id;
  res.json({ id: userId, name: 'Alice' });
});

app.listen(3000);
```

### Fastify

Fastify has built-in TypeScript support:

```typescript
import Fastify from 'fastify';

const server = Fastify();

server.get('/ping', async (request, reply) => {
  return { pong: 'it worked!' };
});

server.listen({ port: 3000 });
```

### NestJS

A TypeScript-first Node.js framework (Angular-inspired):

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  findAll(): string {
    return 'This returns all users';
  }
}
```

Built for TypeScript. Decorators, dependency injection, modular architecture.

### Prisma

Type-safe ORM with generated types:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

const user = await prisma.user.findUnique({
  where: { id: 1 }
});
// user is fully typed based on your schema
```

Prisma generates TypeScript types from your database schema. Changes to the schema automatically update types.

### GraphQL

Type-safe GraphQL clients and servers:

**Apollo Client:**
```bash
npm install @apollo/client graphql
```

**GraphQL Code Generator:**
```bash
npm install --save-dev @graphql-codegen/cli
```

Generates TypeScript types from GraphQL schemas:

```yaml
# codegen.yml
schema: 'http://localhost:4000/graphql'
documents: 'src/**/*.graphql'
generates:
  src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
```

## Build Tools

### Vite

Modern, fast, zero-config TypeScript support:

```bash
npm create vite@latest my-app -- --template vanilla-ts
```

Just works. No tsconfig tweaking needed (mostly).

### Webpack

Mature, configurable, requires setup:

```bash
npm install --save-dev typescript ts-loader
```

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  }
};
```

### esbuild

Extremely fast bundler and transpiler:

```bash
npm install --save-dev esbuild
```

```bash
esbuild src/index.ts --bundle --outfile=dist/index.js
```

**Important:** esbuild doesn't type-check. Run `tsc --noEmit` separately.

### Rollup

Flexible bundler, often used for libraries:

```bash
npm install --save-dev rollup @rollup/plugin-typescript
```

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';

export default {
  input: 'src/index.ts',
  output: {
    file: 'dist/index.js',
    format: 'esm'
  },
  plugins: [typescript()]
};
```

### Turbopack (Next.js)

Next.js 13+ uses Turbopack (Rust-based). TypeScript support is built-in:

```bash
npx create-next-app@latest --typescript
```

### Parcel

Zero-config bundler:

```bash
npm install --save-dev parcel
```

```bash
parcel src/index.html
```

Detects TypeScript automatically. No config needed.

## Testing Frameworks

### Jest

Popular, mature, requires configuration:

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

```typescript
// math.test.ts
import { add } from './math';

test('adds numbers', () => {
  expect(add(2, 3)).toBe(5);
});
```

### Vitest

Modern, Vite-powered, zero-config:

```bash
npm install --save-dev vitest
```

```typescript
// math.test.ts
import { describe, it, expect } from 'vitest';
import { add } from './math';

describe('add', () => {
  it('adds numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

TypeScript support built-in. Extremely fast.

### Testing Library

UI testing with full TypeScript support:

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

```typescript
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button', () => {
  render(<Button label="Click me" onClick={() => {}} />);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});
```

### Playwright

End-to-end testing with TypeScript:

```bash
npm install --save-dev @playwright/test
```

```typescript
import { test, expect } from '@playwright/test';

test('homepage has title', async ({ page }) => {
  await page.goto('http://localhost:3000');
  await expect(page).toHaveTitle(/My App/);
});
```

### Cypress

```bash
npm install --save-dev cypress
```

```typescript
// cypress/e2e/spec.cy.ts
describe('My App', () => {
  it('visits the homepage', () => {
    cy.visit('http://localhost:3000');
    cy.contains('Welcome');
  });
});
```

## Linters and Formatters

### ESLint

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
  ]
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

### Biome

Modern all-in-one linter/formatter (Rust-based):

```bash
npm install --save-dev @biomejs/biome
```

```bash
npx biome check .
npx biome format --write .
```

Extremely fast. Alternative to ESLint + Prettier.

## Runtime Validation

TypeScript types are compile-time only. Runtime validation requires separate tools:

### Zod

Schema validation with TypeScript inference:

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  age: z.number()
});

type User = z.infer<typeof UserSchema>;
// { id: string; name: string; age: number; }

const data = JSON.parse(apiResponse);
const user = UserSchema.parse(data); // Throws if invalid
```

### Yup

Similar to Zod:

```typescript
import * as yup from 'yup';

const userSchema = yup.object({
  name: yup.string().required(),
  age: yup.number().positive().required()
});

await userSchema.validate({ name: 'Alice', age: 30 });
```

### io-ts

Functional approach:

```typescript
import * as t from 'io-ts';

const User = t.type({
  id: t.string,
  name: t.string,
  age: t.number
});

type User = t.TypeOf<typeof User>;
```

### AJV

JSON Schema validator:

```typescript
import Ajv from 'ajv';

const ajv = new Ajv();
const schema = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    age: { type: 'number' }
  },
  required: ['name', 'age']
};

const validate = ajv.compile(schema);
const valid = validate(data);
```

## Documentation

### TSDoc

JSDoc for TypeScript:

```typescript
/**
 * Calculates the sum of two numbers.
 * @param a - The first number
 * @param b - The second number
 * @returns The sum of a and b
 * @example
 * ```ts
 * add(2, 3); // 5
 * ```
 */
function add(a: number, b: number): number {
  return a + b;
}
```

VSCode shows this in IntelliSense.

### TypeDoc

Generate docs from TypeScript code:

```bash
npm install --save-dev typedoc
```

```bash
npx typedoc --out docs src/index.ts
```

Creates HTML documentation from your types and TSDoc comments.

## Package Managers

### npm

Standard. Ships with Node.js:

```bash
npm install typescript
```

### pnpm

Fast, disk-efficient:

```bash
pnpm install typescript
```

Uses symlinks, saves disk space, faster installs.

### Yarn

Alternative to npm:

```bash
yarn add typescript
```

Yarn 2+ (Berry) has workspaces, Plug'n'Play mode.

### Bun

Extremely fast (native code):

```bash
bun install typescript
```

Also runs TypeScript natively (`bun run script.ts`).

## Monorepo Tools

### Turborepo

Fast build system for monorepos:

```bash
npx create-turbo@latest
```

Caches builds, runs tasks in parallel. TypeScript-aware.

### Nx

Powerful monorepo tool:

```bash
npx create-nx-workspace
```

Supports TypeScript out of the box. Dependency graph, affected commands, caching.

### Lerna

Classic monorepo manager:

```bash
npx lerna init
```

Less popular now. Turborepo and Nx are more modern.

## Deployment

### Vercel

Zero-config deployments for TypeScript:

```bash
npm install -g vercel
vercel
```

Supports Next.js, Vite, vanilla TypeScript. Detects TypeScript automatically.

### Netlify

Similar to Vercel:

```bash
npm install -g netlify-cli
netlify deploy
```

### Deno Deploy

Deploy TypeScript directly to the edge:

```bash
deployctl deploy --project=my-project server.ts
```

No build step. TypeScript runs in production.

### Cloudflare Workers

Supports TypeScript via Wrangler:

```bash
npm install -g wrangler
wrangler init
```

```typescript
// worker.ts
export default {
  async fetch(request: Request): Promise<Response> {
    return new Response('Hello from TypeScript!');
  }
};
```

### AWS Lambda

Use AWS CDK or Serverless Framework:

```bash
npm install -g aws-cdk
cdk init app --language typescript
```

## Editor Support

### VS Code

Best-in-class TypeScript support. Built by the same team (Microsoft).

**Extensions:**
- ESLint
- Prettier
- Error Lens (inline errors)
- Pretty TypeScript Errors (better error messages)

### WebStorm

Full IDE with built-in TypeScript support. No extensions needed.

### Neovim/Vim

Use `coc.nvim` or native LSP with `typescript-language-server`:

```bash
npm install -g typescript-language-server
```

### Emacs

Use `lsp-mode` or `tide`:

```bash
npm install -g typescript-language-server
```

## Community Resources

### TypeScript Handbook

Official docs: [typescriptlang.org/docs](https://www.typescriptlang.org/docs/)

Comprehensive. Start here.

### TypeScript Deep Dive

Free online book by Basarat Ali Syed. Practical, example-driven.

### Effective TypeScript

Book by Dan Vanderkam. 62 specific tips for better TypeScript.

### Total TypeScript

Video course by Matt Pocock. Interactive, exercise-based.

### TypeScript Playground

[typescriptlang.org/play](https://www.typescriptlang.org/play)

Test code snippets, share examples, explore compiler options.

### GitHub Issues

[github.com/microsoft/TypeScript/issues](https://github.com/microsoft/TypeScript/issues)

Report bugs, request features, see what's coming.

### Twitter/X

Follow:
- @typescript
- @mattpocockuk (Matt Pocock)
- @drosenwasser (Daniel Rosenwasser, TypeScript PM)

## Emerging Patterns

### Type-Safe SQL

**Kysely:**
```typescript
const result = await db
  .selectFrom('users')
  .select(['id', 'name'])
  .where('age', '>', 18)
  .execute();
// result is typed based on schema
```

**Drizzle:**
```typescript
const users = await db.select().from(usersTable).where(eq(usersTable.age, 30));
// Fully typed
```

### Type-Safe APIs

**tRPC:**
```typescript
const user = await trpc.user.getById.query({ id: '123' });
// user is typed, no code generation needed
```

Full-stack type safety. Client knows server types automatically.

### Effect-TS

Functional programming library with powerful type system:

```typescript
import { Effect } from 'effect';

const program = Effect.gen(function* (_) {
  const user = yield* _(fetchUser('123'));
  return user.name;
});
```

Advanced. Steep learning curve. Growing community.

## What You've Learned

- **Frameworks** (React, Vue, Angular, Svelte) all support TypeScript
- **Build tools** (Vite, esbuild, Webpack) handle TypeScript transpilation
- **Testing frameworks** (Vitest, Jest, Playwright) work seamlessly with TypeScript
- **Runtime validation** (Zod, Yup) bridges compile-time and runtime types
- **Documentation** tools (TSDoc, TypeDoc) extract types into docs
- **Monorepo tools** (Turborepo, Nx) scale TypeScript across packages
- **Deployment platforms** (Vercel, Netlify, Deno Deploy) support TypeScript
- **Community resources** (handbook, playground, courses) are extensive

The TypeScript ecosystem is mature. Most tools you already use support it. New tools are being built TypeScript-first.

---

**Next:** [Chapter 11: Migrating JavaScript to TypeScript](./11-migration.md)
