# Chapter 12: TypeScript in the Wild (React, Node, and Beyond)

You've learned TypeScript. Now let's see it in context—the frameworks and libraries where TypeScript really earns its keep.

This chapter covers the most common real-world scenarios: React, Node.js, and other popular stacks.

## React + TypeScript

React and TypeScript are a natural fit. Components have clear inputs (props) and outputs (JSX). TypeScript makes this explicit.

### Setup

```bash
# Create React App (legacy)
npx create-react-app my-app --template typescript

# Vite (modern, recommended)
npm create vite@latest my-app -- --template react-ts

# Next.js
npx create-next-app@latest --typescript
```

### Functional Components

```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
}

function Button({ label, onClick, disabled = false, variant = 'primary' }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {label}
    </button>
  );
}
```

Or with `React.FC` (not recommended, but common):

```typescript
const Button: React.FC<ButtonProps> = ({ label, onClick, disabled = false, variant = 'primary' }) => {
  return <button onClick={onClick} disabled={disabled} className={`btn btn-${variant}`}>{label}</button>;
};
```

**Why not `React.FC`?**
- Makes children implicit (you might not want children)
- Prevents return type inference
- Community is moving away from it

### Props with Children

```typescript
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div>{children}</div>
    </div>
  );
}
```

`React.ReactNode` is the type for anything renderable (elements, strings, numbers, fragments, etc.).

### Event Handlers

```typescript
function Form() {
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    // Handle form submission
  };

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    console.log(event.target.value);
  };

  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Button clicked');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
      <button onClick={handleClick}>Submit</button>
    </form>
  );
}
```

Common event types:
- `React.MouseEvent<T>`
- `React.KeyboardEvent<T>`
- `React.ChangeEvent<T>`
- `React.FormEvent<T>`
- `React.FocusEvent<T>`

### Hooks

#### useState

```typescript
const [count, setCount] = useState(0); // Inferred as number
const [name, setName] = useState(''); // Inferred as string

// Explicit type
const [user, setUser] = useState<User | null>(null);
```

#### useRef

```typescript
const inputRef = useRef<HTMLInputElement>(null);

useEffect(() => {
  inputRef.current?.focus();
}, []);

return <input ref={inputRef} />;
```

#### useReducer

```typescript
interface State {
  count: number;
}

type Action = { type: 'increment' } | { type: 'decrement' } | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

#### Custom Hooks

```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}

// Usage
const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
```

The `as const` ensures the return type is `[T, Dispatch<SetStateAction<T>>]` instead of `(T | Dispatch<SetStateAction<T>>)[]`.

### Context

```typescript
interface AuthContextValue {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | undefined>(undefined);

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const user = await apiLogin(email, password);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

### Higher-Order Components (Legacy)

```typescript
function withLoading<P extends object>(Component: React.ComponentType<P>) {
  return function WithLoadingComponent({ isLoading, ...props }: P & { isLoading: boolean }) {
    if (isLoading) return <div>Loading...</div>;
    return <Component {...(props as P)} />;
  };
}

const ButtonWithLoading = withLoading(Button);
```

HOCs are less common now. Hooks replaced most use cases.

### Generic Components

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage
<List
  items={[1, 2, 3]}
  renderItem={(num) => <span>{num * 2}</span>}
/>
```

### React Router

```typescript
import { useParams, useNavigate } from 'react-router-dom';

interface RouteParams {
  id: string;
}

function UserProfile() {
  const { id } = useParams<RouteParams>();
  const navigate = useNavigate();

  useEffect(() => {
    fetchUser(id);
  }, [id]);

  return <div>User {id}</div>;
}
```

## Node.js + TypeScript

### Setup

```bash
npm init -y
npm install --save-dev typescript @types/node
npx tsc --init
```

**tsconfig.json for Node:**

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
    "moduleResolution": "node",
    "resolveJsonModule": true
  },
  "include": ["src/**/*"]
}
```

### Express

```typescript
import express, { Request, Response, NextFunction } from 'express';

const app = express();

app.use(express.json());

interface CreateUserRequest {
  name: string;
  email: string;
}

app.post('/users', (req: Request<{}, {}, CreateUserRequest>, res: Response) => {
  const { name, email } = req.body;
  // name and email are typed
  res.json({ id: '123', name, email });
});

// Typed params
app.get('/users/:id', (req: Request<{ id: string }>, res: Response) => {
  const { id } = req.params;
  res.json({ id, name: 'Alice' });
});

// Error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Fastify

```typescript
import Fastify from 'fastify';

const server = Fastify();

interface CreateUserBody {
  name: string;
  email: string;
}

server.post<{ Body: CreateUserBody }>('/users', async (request, reply) => {
  const { name, email } = request.body;
  return { id: '123', name, email };
});

server.listen({ port: 3000 });
```

### NestJS (TypeScript-first)

```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';

interface CreateUserDto {
  name: string;
  email: string;
}

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    return { id, name: 'Alice' };
  }

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return { id: '123', ...createUserDto };
  }
}
```

NestJS uses decorators heavily. Enable `experimentalDecorators: true` in `tsconfig.json`.

### File System Operations

```typescript
import { promises as fs } from 'fs';
import * as path from 'path';

async function readConfig(): Promise<{ port: number; host: string }> {
  const configPath = path.join(__dirname, 'config.json');
  const content = await fs.readFile(configPath, 'utf-8');
  return JSON.parse(content);
}
```

### Working with Streams

```typescript
import { createReadStream } from 'fs';
import { pipeline } from 'stream/promises';
import { createGzip } from 'zlib';

async function compressFile(inputPath: string, outputPath: string) {
  await pipeline(
    createReadStream(inputPath),
    createGzip(),
    fs.createWriteStream(outputPath)
  );
}
```

## Database Integration

### Prisma (Type-safe ORM)

```bash
npm install @prisma/client
npm install --save-dev prisma
npx prisma init
```

**schema.prisma:**

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

**Generated TypeScript:**

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  const user = await prisma.user.create({
    data: {
      email: 'alice@example.com',
      name: 'Alice',
      posts: {
        create: { title: 'Hello World' }
      }
    }
  });
  // user is fully typed based on schema
}
```

### TypeORM

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}

// Usage
const userRepository = dataSource.getRepository(User);
const user = await userRepository.findOne({ where: { id: 1 } });
```

## GraphQL

### Apollo Server

```typescript
import { ApolloServer, gql } from 'apollo-server';

const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
  }

  type Query {
    user(id: ID!): User
    users: [User!]!
  }
`;

interface User {
  id: string;
  name: string;
  email: string;
}

const resolvers = {
  Query: {
    user: (_: unknown, { id }: { id: string }): User => {
      return { id, name: 'Alice', email: 'alice@example.com' };
    },
    users: (): User[] => {
      return [{ id: '1', name: 'Alice', email: 'alice@example.com' }];
    }
  }
};

const server = new ApolloServer({ typeDefs, resolvers });
server.listen().then(({ url }) => console.log(`Server ready at ${url}`));
```

### GraphQL Code Generator

Auto-generate types from schema:

```bash
npm install --save-dev @graphql-codegen/cli @graphql-codegen/typescript
```

**codegen.yml:**

```yaml
schema: 'http://localhost:4000/graphql'
documents: 'src/**/*.graphql'
generates:
  src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
```

Now queries are fully typed:

```typescript
import { useQuery } from '@apollo/client';
import { GetUserQuery, GetUserQueryVariables } from './generated/graphql';

const { data } = useQuery<GetUserQuery, GetUserQueryVariables>(GET_USER, {
  variables: { id: '123' }
});

console.log(data?.user?.name); // Fully typed
```

## Testing

### Jest with TypeScript

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

// math.test.ts
import { add } from './math';

describe('add', () => {
  it('adds two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });

  it('handles negative numbers', () => {
    expect(add(-1, 1)).toBe(0);
  });
});
```

### React Testing Library

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

test('button click triggers callback', () => {
  const handleClick = jest.fn();
  render(<Button label="Click me" onClick={handleClick} />);

  const button = screen.getByText('Click me');
  fireEvent.click(button);

  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

### Type-Safe Mocks

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

const mockUser: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
};

jest.mock('./api', () => ({
  fetchUser: jest.fn((): Promise<User> => Promise.resolve(mockUser))
}));
```

## Deno in Production

Remember, Deno runs TypeScript natively:

```typescript
// server.ts
import { serve } from "https://deno.land/std@0.208.0/http/server.ts";

interface User {
  id: string;
  name: string;
}

serve(async (req: Request) => {
  const url = new URL(req.url);

  if (url.pathname === '/users') {
    const users: User[] = [{ id: '1', name: 'Alice' }];
    return new Response(JSON.stringify(users), {
      headers: { 'content-type': 'application/json' }
    });
  }

  return new Response('Not found', { status: 404 });
});
```

```bash
deno run --allow-net server.ts
```

No build step. TypeScript in production.

## Best Practices (Real-World)

### 1. Co-locate Types with Code

```typescript
// Bad: types in separate file
// types.ts
export interface User { /* ... */ }

// user.ts
import { User } from './types';

// Good: types with implementation
// user.ts
export interface User { /* ... */ }
export function createUser(name: string): User { /* ... */ }
```

### 2. Use Branded Types for IDs

```typescript
type UserId = string & { __brand: 'UserId' };
type PostId = string & { __brand: 'PostId' };

function getUser(id: UserId) { /* ... */ }
function getPost(id: PostId) { /* ... */ }

const userId = 'user-123' as UserId;
const postId = 'post-456' as PostId;

getUser(userId); // OK
getUser(postId); // Error
```

### 3. Prefer Unknown Over Any

```typescript
// Bad
function parse(data: any) {
  return data.value;
}

// Good
function parse(data: unknown): number {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return (data as { value: number }).value;
  }
  throw new Error('Invalid data');
}
```

### 4. Use Exhaustive Checks

```typescript
type Status = 'pending' | 'approved' | 'rejected';

function handleStatus(status: Status) {
  switch (status) {
    case 'pending':
      return 'Waiting...';
    case 'approved':
      return 'Approved!';
    case 'rejected':
      return 'Rejected.';
    default:
      const exhaustive: never = status;
      throw new Error(`Unhandled status: ${exhaustive}`);
  }
}
```

### 5. Avoid Enums (Use Union Types)

```typescript
// Bad
enum Status {
  Pending = 'PENDING',
  Approved = 'APPROVED',
  Rejected = 'REJECTED'
}

// Good
type Status = 'pending' | 'approved' | 'rejected';
```

Enums have runtime overhead. Union types are just types.

## What You've Learned

- **React + TypeScript** for type-safe components, props, hooks
- **Node.js + TypeScript** with Express, Fastify, NestJS
- **Database integration** with Prisma, TypeORM
- **GraphQL** with typed queries and mutations
- **Testing** with Jest, Testing Library, type-safe mocks
- **Deno** for production TypeScript without build steps
- **Best practices** for real-world TypeScript

TypeScript transforms how you build applications. Types catch bugs, improve refactoring, and make large codebases manageable.

You're no longer a JavaScript developer learning TypeScript. You're a TypeScript developer who understands JavaScript.

---

## Conclusion

You started this book as an experienced JavaScript developer. You knew closures, prototypes, async/await, and the module system. You'd shipped code.

Now you understand TypeScript: its type system, toolchain, advanced features, ecosystem, and real-world application. You know when to use strict types, when to use `any`, when to fight the compiler, and when to trust it.

TypeScript isn't a silver bullet. It won't eliminate bugs. It won't make bad code good. But it will catch entire classes of errors at compile time, make refactoring safer, and encode your design intent in a way the computer can verify.

The JavaScript you write won't change much. The confidence you have in it will.

**Welcome to TypeScript.** Now go build something.
