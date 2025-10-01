---
layout: chapter
title: "Migrating JavaScript to TypeScript"
chapter_number: 11
permalink: /chapters/migration/
---

# Chapter 11: Migrating JavaScript to TypeScript

You have a JavaScript codebase. Maybe 10,000 lines. Maybe 100,000. Maybe more. Rewriting it in TypeScript isn't realistic. Ignoring TypeScript means missing its benefits.

The solution: **incremental migration**. TypeScript was designed for this. You can adopt it gradually, file by file, module by module, without breaking your app.

This chapter is your migration playbook.

## The Strategy

**Don't rewrite. Gradually type.**

TypeScript's philosophy: **JavaScript is valid TypeScript**. Start with `.js` files, rename them to `.ts`, fix errors incrementally.

### Migration Stages

1. **Setup** - Add TypeScript to your project
2. **Allowlist** - Enable `allowJs`, keep existing JS files running
3. **Incremental conversion** - Rename files to `.ts` one at a time
4. **Strict mode** - Tighten compiler options as you go
5. **Complete coverage** - All files typed, strict mode enabled

This can take weeks or months. That's fine. Incremental progress beats paralysis.

## Stage 1: Setup

### Install TypeScript

```bash
npm install --save-dev typescript @types/node
```

(Add other `@types/*` packages as needed: `@types/react`, `@types/express`, etc.)

### Initialize tsconfig.json

```bash
npx tsc --init
```

Edit it for migration:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],

    "allowJs": true,           // Allow .js files
    "checkJs": false,          // Don't type-check .js files yet
    "outDir": "./dist",
    "rootDir": "./src",

    "strict": false,           // Start loose, tighten later
    "noImplicitAny": false,    // Allow implicit any
    "strictNullChecks": false, // Allow null/undefined freely

    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    "moduleResolution": "node",
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**Key settings:**
- `allowJs: true` - TypeScript compiles `.js` files alongside `.ts`
- `checkJs: false` - Don't type-check JavaScript (yet)
- `strict: false` - Loose mode, fewer errors

### Update package.json Scripts

```json
{
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "type-check": "tsc --noEmit"
  }
}
```

### Test the Build

```bash
npm run build
```

If it compiles, you're ready. If it doesn't, check:
- File paths in `include`/`exclude`
- Missing `@types/*` packages
- Syntax errors in `.js` files (TypeScript is stricter than some JS runtimes)

## Stage 2: Allowlist (Keep Everything Working)

Your goal: **TypeScript compiles your project without changes.**

**Don't rename files yet.** Just get the build working.

### Common Issues

#### Missing Type Definitions

```
Error: Could not find a declaration file for module 'lodash'.
```

**Fix:**
```bash
npm install --save-dev @types/lodash
```

#### Global Variables

If you have global scripts (e.g., `gtag`, `analytics`):

```typescript
// src/globals.d.ts
declare const gtag: (...args: any[]) => void;
declare const analytics: {
  track(event: string): void;
};
```

#### Dynamic requires

```javascript
const module = require(dynamicPath); // Error
```

TypeScript can't type this. Options:
- Use `import()` instead (if possible)
- Add `// @ts-ignore` to suppress the error
- Refactor to static imports

### Verify Everything Works

```bash
npm run build
node dist/index.js
```

If your app runs as before, proceed.

## Stage 3: Incremental Conversion

Now the real work begins. Convert files one by one.

### Start with Low-Risk Files

**Good first candidates:**
- Utility functions (pure functions, no side effects)
- Constants and configuration
- Type definitions (interfaces, types)
- New code (anything you're about to write)

**Bad first candidates:**
- Core business logic (high complexity, high risk)
- Files with many dependencies (errors cascade)
- Code you don't understand (fix bugs first)

### Rename .js → .ts

```bash
mv src/utils/math.js src/utils/math.ts
```

Run the compiler:

```bash
npm run type-check
```

You'll get errors. That's expected.

### Fix Errors (The Four Strategies)

#### 1. Add Type Annotations

```typescript
// Before (JS)
function add(a, b) {
  return a + b;
}

// After (TS)
function add(a: number, b: number): number {
  return a + b;
}
```

#### 2. Use `any` (Tactical Retreat)

```typescript
function processData(data: any) {
  // Too complex to type right now
  return data.map(/* ... */);
}
```

`any` lets you defer typing. Come back later.

#### 3. Suppress Errors with `@ts-ignore`

```typescript
// @ts-ignore
const result = complexLegacyCode();
```

Use sparingly. Prefer fixing the issue or using `any`.

#### 4. Fix the Bug

Sometimes TypeScript catches real bugs:

```javascript
// Before
if (user.age = 18) { // Assignment instead of comparison!
  // ...
}
```

```typescript
// After
if (user.age === 18) {
  // ...
}
```

TypeScript errors often reveal latent bugs.

### Repeat

Convert files one at a time. Commit after each file. If something breaks, revert.

### Track Progress

Create a checklist:

```markdown
## Migration Progress

- [x] utils/math.ts
- [x] utils/string.ts
- [ ] services/api.ts
- [ ] components/Button.tsx
- [ ] ...
```

Or use a script:

```bash
find src -name "*.js" | wc -l  # JS files remaining
find src -name "*.ts" | wc -l  # TS files done
```

## Stage 4: Tighten Compiler Options

As you convert more files, enable stricter checks.

### Enable `noImplicitAny`

```json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```

Now TypeScript errors on implicit `any`:

```typescript
function greet(name) { // Error: Parameter 'name' implicitly has an 'any' type
  console.log(`Hello, ${name}`);
}
```

Fix by adding types:

```typescript
function greet(name: string) {
  console.log(`Hello, ${name}`);
}
```

### Enable `strictNullChecks`

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

Now `null` and `undefined` must be handled explicitly:

```typescript
function getUser(id: string): User | null {
  // ...
}

const user = getUser('123');
console.log(user.name); // Error: Object is possibly 'null'

// Fix:
if (user) {
  console.log(user.name);
}
```

### Enable Full `strict` Mode

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

Enables all strict options:
- `noImplicitAny`
- `strictNullChecks`
- `strictFunctionTypes`
- `strictBindCallApply`
- `strictPropertyInitialization`
- `noImplicitThis`
- `alwaysStrict`

This is the end goal. Don't enable it too early, or you'll drown in errors.

## Stage 5: Complete Coverage

All files are `.ts`. Strict mode is enabled. Now refine:

### Remove `any` Types

Search for `any`:

```bash
grep -r ": any" src/
```

Replace with proper types:

```typescript
// Before
function process(data: any) {
  return data.map((item: any) => item.id);
}

// After
interface Item {
  id: string;
}

function process(data: Item[]): string[] {
  return data.map(item => item.id);
}
```

### Remove `@ts-ignore` Comments

Each `@ts-ignore` is technical debt. Fix or refactor:

```typescript
// Before
// @ts-ignore
const result = legacyFunction();

// After (add types for legacyFunction)
interface LegacyResult {
  // ...
}

declare function legacyFunction(): LegacyResult;
const result = legacyFunction();
```

### Add Runtime Validation

TypeScript types don't validate runtime data (API responses, user input). Add validation:

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email()
});

const response = await fetch('/api/user');
const data = await response.json();
const user = UserSchema.parse(data); // Throws if invalid
```

Now your types match runtime reality.

## Common Migration Challenges

### Challenge 1: Large Files

**Problem:** A 2,000-line file with 500 errors.

**Solution:** Split it first, then type:

```bash
# Extract utilities
mv src/bigFile.js src/bigFile.js.bak
mkdir src/bigFile
# Split into smaller modules
# Then convert each small file to .ts
```

### Challenge 2: Untyped Dependencies

**Problem:** A library has no `@types/*` package.

**Solution:** Write your own `.d.ts`:

```typescript
// src/types/my-library.d.ts
declare module 'my-library' {
  export function doSomething(x: string): number;
}
```

Or use `any`:

```typescript
declare module 'my-library';
```

### Challenge 3: Dynamic Code

**Problem:** Heavy use of `eval`, dynamic property access, runtime type manipulation.

**Solution:** TypeScript can't type truly dynamic code. Options:
- Refactor to static patterns
- Use `any` for dynamic sections
- Add runtime checks with type guards

### Challenge 4: Team Resistance

**Problem:** Team members don't want to learn TypeScript.

**Solution:**
- Start with new code only (no migration pressure)
- Pair program (teach by doing)
- Show value (catch real bugs, better autocomplete)
- Don't force strict mode until everyone's comfortable

### Challenge 5: Build Performance

**Problem:** TypeScript slows down builds.

**Solution:**
- Enable `skipLibCheck: true`
- Use `incremental: true`
- Split monoliths into smaller modules
- Use project references for monorepos
- Consider Deno/Bun for faster execution (no build step)

## Migration Tools

### ts-migrate (Airbnb) - Legacy

**Note:** ts-migrate is no longer actively maintained (last updated 2021).

```bash
npx ts-migrate migrate src/
```

Renames files, adds basic types, inserts `@ts-ignore` where needed. Still works but consider alternatives:

- **JSDoc to TypeScript:** Use TypeScript's built-in JSDoc conversion
- **Incremental `checkJs`:** Enable `checkJs` with `@ts-check` comments
- **Manual migration:** Often the most effective for learning

### TypeStat

Automated type inference from runtime behavior:

```bash
npm install -g typestat
typestat --config typestat.json
```

Analyzes code, adds types. Experimental but useful.

### Manual is Often Better

Automated tools help, but manual migration teaches you TypeScript and your codebase. Don't skip the learning. For most projects in 2025, a gradual manual migration with `allowJs` is the most reliable approach.

## Progressive Enhancement Strategy

**Start loose, tighten gradually:**

```json
// Week 1: Basic setup
{
  "strict": false,
  "noImplicitAny": false,
  "strictNullChecks": false
}

// Week 4: Enable noImplicitAny
{
  "strict": false,
  "noImplicitAny": true,
  "strictNullChecks": false
}

// Week 8: Enable strictNullChecks
{
  "strict": false,
  "noImplicitAny": true,
  "strictNullChecks": true
}

// Week 12: Full strict mode
{
  "strict": true
}
```

Adjust timeline to your team's capacity.

## When to Migrate

**Good reasons:**
- You're adding new features (perfect time to type new code)
- You're refactoring (already touching the code)
- You're experiencing runtime errors that types would catch
- You're onboarding new developers (types are documentation)

**Bad reasons:**
- "Everyone else is doing it"
- "Management said so"
- "I read a blog post"

Migrate when the value justifies the cost.

## When NOT to Migrate

**Skip TypeScript if:**
- Small scripts (< 500 lines)
- Prototype/throwaway code
- Your team doesn't want it (forcing it causes resentment)
- External constraints (build tools don't support it)

TypeScript is optional. Use it when it helps.

## Measuring Success

Track these metrics:

**Quantitative:**
- % of files typed (`.ts` vs `.js`)
- % of code typed (lines with types vs `any`)
- # of `@ts-ignore` comments (lower is better)
- # of strict compiler options enabled
- Runtime errors caught by types (before deploy)

**Qualitative:**
- Developer confidence (do they trust the types?)
- Onboarding speed (new devs ramp up faster?)
- Refactoring ease (large changes less scary?)

## What You've Learned

- **Migration is incremental** - File by file, not all at once
- **Start loose, tighten gradually** - `allowJs` → convert files → enable strict
- **Use tactical retreats** - `any` and `@ts-ignore` are temporary tools
- **Low-risk files first** - Utils, new code, simple modules
- **Tooling helps** - ts-migrate, automated refactors
- **Team buy-in matters** - Force nothing, teach by example
- **Measure progress** - Track files, errors, coverage

Migration isn't a weekend project. It's a journey. Done well, it pays dividends for years.

---

**Next:** [Chapter 12: TypeScript in the Wild (React, Node, and Beyond)](./12-typescript-in-the-wild.md)
