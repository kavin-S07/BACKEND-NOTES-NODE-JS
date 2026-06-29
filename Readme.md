# 🚀 Express.js + Drizzle ORM + PostgreSQL Starter

A production-ready REST API boilerplate using **Express.js**, **Drizzle ORM**, and **PostgreSQL** — type-safe, lightweight, and built for developer velocity.

---

## 📖 Table of Contents

- [What is Drizzle ORM?](#what-is-drizzle-orm)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Database Setup](#database-setup)
- [Schema Definition](#schema-definition)
- [Migrations](#migrations)
- [CRUD Operations](#crud-operations)
- [Repository Pattern](#repository-pattern)
- [Express Routes](#express-routes)
- [Running the Project](#running-the-project)
- [NPM Scripts](#npm-scripts)
- [Testing & Debugging](#testing--debugging)
- [FAQ & Troubleshooting](#faq--troubleshooting)

---

## What is Drizzle ORM?

Drizzle ORM is a **TypeScript-first**, **SQL-like** ORM for Node.js. Unlike Prisma (which abstracts SQL heavily) or raw `pg` (which gives no type safety), Drizzle sits in the sweet spot: you write queries that feel like SQL, but get full TypeScript inference throughout.

### Core Concepts

| Concept | What It Means |
|---|---|
| **Schema** | Define tables in TypeScript — Drizzle infers all types from them |
| **Query Builder** | Chain methods (`.select()`, `.where()`, `.join()`) to build SQL |
| **Migrations** | Drizzle generates SQL migration files from your schema changes |
| **`drizzle-kit`** | CLI tool for generating, running, and inspecting migrations |

### Why Drizzle Over Alternatives?

- **Zero runtime overhead** — queries compile to raw SQL, no magic at runtime
- **Full TypeScript inference** — your schema IS your types; no separate type generation step
- **SQL transparency** — you always know what SQL will be executed
- **Lightweight** — no binary dependencies, no Rust engine, no Docker requirement for migrations
- **Familiar syntax** — if you know SQL, you already know 80% of Drizzle

---

## Prerequisites

| Tool | Minimum Version | Notes |
|---|---|---|
| Node.js | 18.x | LTS recommended |
| npm | 9.x | or pnpm / yarn |
| PostgreSQL | 14.x | Running locally or via Docker |
| TypeScript | 5.x | Strict mode recommended |

Install PostgreSQL locally or run it via Docker:

```bash
docker run --name pg-dev -e POSTGRES_PASSWORD=secret -e POSTGRES_DB=myapp -p 5432:5432 -d postgres:16
```

---

## Project Structure

```
my-app/
├── src/
│   ├── db/
│   │   ├── index.ts          # Database connection
│   │   └── schema.ts         # Table definitions
│   ├── repositories/
│   │   └── userRepository.ts # Data access layer
│   ├── routes/
│   │   └── users.ts          # Express route handlers
│   └── index.ts              # App entry point
├── drizzle/                  # Auto-generated migration files
├── drizzle.config.ts         # Drizzle Kit configuration
├── .env
├── package.json
└── tsconfig.json
```

---

## Installation

### 1. Initialize the project

```bash
mkdir my-app && cd my-app
npm init -y
```

### 2. Install dependencies

```bash
# Runtime dependencies
npm install express drizzle-orm pg dotenv

# Type definitions
npm install -D @types/express @types/pg typescript ts-node tsx

# Drizzle CLI (for migrations)
npm install -D drizzle-kit
```

### 3. Configure TypeScript

```bash
npx tsc --init
```

Update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "skipLibCheck": true
  },
  "include": ["src/**/*", "drizzle.config.ts"]
}
```

---

## Environment Variables

Create a `.env` file in the project root:

```env
# Database
DATABASE_URL=postgresql://postgres:secret@localhost:5432/myapp

# App
PORT=3000
NODE_ENV=development
```

> ⚠️ Never commit `.env` to version control. Add it to `.gitignore`.

---

## Database Setup

### `src/db/index.ts`

```typescript
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";
import * as schema from "./schema";
import dotenv from "dotenv";

dotenv.config();

// Create a connection pool (recommended for production)
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,               // max pool connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Export the Drizzle instance — use this everywhere in your app
export const db = drizzle(pool, { schema });
export type DB = typeof db;
```

---

## Schema Definition

### `src/db/schema.ts`

```typescript
import {
  pgTable,
  serial,
  text,
  varchar,
  integer,
  boolean,
  timestamp,
  pgEnum,
} from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";

// ─── Enums ───────────────────────────────────────────────────────────────────
export const roleEnum = pgEnum("role", ["user", "admin", "manager"]);

// ─── Users Table ─────────────────────────────────────────────────────────────
export const users = pgTable("users", {
  id:        serial("id").primaryKey(),
  name:      varchar("name", { length: 100 }).notNull(),
  email:     varchar("email", { length: 255 }).notNull().unique(),
  role:      roleEnum("role").default("user").notNull(),
  age:       integer("age"),
  isActive:  boolean("is_active").default(true).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// ─── Posts Table ─────────────────────────────────────────────────────────────
export const posts = pgTable("posts", {
  id:        serial("id").primaryKey(),
  title:     text("title").notNull(),
  body:      text("body"),
  userId:    integer("user_id").references(() => users.id, { onDelete: "cascade" }).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

// ─── Relations (for join inference) ──────────────────────────────────────────
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.userId], references: [users.id] }),
}));

// ─── TypeScript Types ─────────────────────────────────────────────────────────
export type User        = typeof users.$inferSelect;   // SELECT result type
export type NewUser     = typeof users.$inferInsert;   // INSERT input type
export type Post        = typeof posts.$inferSelect;
export type NewPost     = typeof posts.$inferInsert;
```

> 💡 **Key insight:** `$inferSelect` and `$inferInsert` give you fully typed models for free — no separate interface definitions needed.

---

## Migrations

### `drizzle.config.ts`

```typescript
import type { Config } from "drizzle-kit";
import dotenv from "dotenv";

dotenv.config();

export default {
  schema: "./src/db/schema.ts",
  out: "./drizzle",               // Where migration SQL files are stored
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

### Migration Commands

```bash
# 1. Generate migration SQL from your schema (run after every schema change)
npx drizzle-kit generate

# 2. Apply migrations to your database
npx drizzle-kit migrate

# 3. Open Drizzle Studio (visual DB browser) in your browser
npx drizzle-kit studio

# 4. Push schema changes directly (dev only — skips migration files)
npx drizzle-kit push
```

> 📌 **Workflow tip:** Use `push` during early development for fast iteration. Switch to `generate` + `migrate` for production-grade version-controlled migrations.

---

## CRUD Operations

All operations are `async` and should be `await`ed. Import `db` from `src/db/index.ts` and operators from `drizzle-orm`.

```typescript
import { db } from "../db";
import { users, posts } from "../db/schema";
import { eq, and, or, gt, like, ilike, inArray, asc, desc, count, sql } from "drizzle-orm";
```

### Create

```typescript
// Insert a single user — returns the inserted row
const [newUser] = await db
  .insert(users)
  .values({ name: "Kavin", email: "kavin@gmail.com", age: 25 })
  .returning();

// Insert multiple users at once
const newUsers = await db
  .insert(users)
  .values([
    { name: "Alice", email: "alice@gmail.com" },
    { name: "Bob",   email: "bob@gmail.com"   },
  ])
  .returning();
```

### Read

```typescript
// Select all users
const allUsers = await db.select().from(users);

// Select specific columns
const names = await db.select({ id: users.id, name: users.name }).from(users);

// Select by ID
const [user] = await db.select().from(users).where(eq(users.id, 1));

// Filter with multiple conditions
const activeAdmins = await db
  .select()
  .from(users)
  .where(and(eq(users.role, "admin"), eq(users.isActive, true)));

// Case-insensitive search
const results = await db
  .select()
  .from(users)
  .where(ilike(users.name, "%kavin%"));

// Pagination
const page = 2;
const pageSize = 10;
const paginated = await db
  .select()
  .from(users)
  .orderBy(asc(users.createdAt))
  .limit(pageSize)
  .offset((page - 1) * pageSize);
```

### Update

```typescript
// Update by ID — always use .where() to avoid updating all rows
const [updated] = await db
  .update(users)
  .set({ name: "John", updatedAt: new Date() })
  .where(eq(users.id, 1))
  .returning();

// Conditional update: deactivate all non-admin users over 60
await db
  .update(users)
  .set({ isActive: false })
  .where(and(gt(users.age, 60), eq(users.role, "user")));
```

### Delete

```typescript
// Delete by ID
const [deleted] = await db
  .delete(users)
  .where(eq(users.id, 1))
  .returning();

// Delete multiple by IDs
await db.delete(users).where(inArray(users.id, [1, 2, 3]));
```

### Joins

```typescript
// Inner join: users with their posts
const usersWithPosts = await db
  .select({
    userId:    users.id,
    userName:  users.name,
    postTitle: posts.title,
  })
  .from(users)
  .innerJoin(posts, eq(users.id, posts.userId));

// Left join: all users, even those without posts
const allUsersWithOptionalPosts = await db
  .select()
  .from(users)
  .leftJoin(posts, eq(users.id, posts.userId));
```

### Aggregates & Grouping

```typescript
// Count users per role
const roleStats = await db
  .select({ role: users.role, total: count() })
  .from(users)
  .groupBy(users.role);

// Average age of active users
const [{ avgAge }] = await db
  .select({ avgAge: sql<number>`avg(${users.age})` })
  .from(users)
  .where(eq(users.isActive, true));
```

### Transactions

```typescript
// Wrap multiple operations — all succeed or all roll back
const result = await db.transaction(async (tx) => {
  const [user] = await tx
    .insert(users)
    .values({ name: "Kavin", email: "kavin@gmail.com" })
    .returning();

  const [post] = await tx
    .insert(posts)
    .values({ title: "Hello World", userId: user.id })
    .returning();

  return { user, post };
});
```

---

## Repository Pattern

Wrap your Drizzle queries in repository classes to keep routes thin and logic reusable.

### `src/repositories/userRepository.ts`

```typescript
import { eq, ilike, and, count } from "drizzle-orm";
import { db } from "../db";
import { users, type User, type NewUser } from "../db/schema";

export class UserRepository {
  // ─── Read ─────────────────────────────────────────────────────────────────

  async findAll(page = 1, limit = 10): Promise<User[]> {
    return db
      .select()
      .from(users)
      .limit(limit)
      .offset((page - 1) * limit)
      .orderBy(users.createdAt);
  }

  async findById(id: number): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.id, id));
    return user;
  }

  async findByEmail(email: string): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.email, email));
    return user;
  }

  async search(query: string): Promise<User[]> {
    return db
      .select()
      .from(users)
      .where(ilike(users.name, `%${query}%`));
  }

  async count(): Promise<number> {
    const [{ total }] = await db.select({ total: count() }).from(users);
    return total;
  }

  // ─── Write ────────────────────────────────────────────────────────────────

  async create(data: NewUser): Promise<User> {
    const [user] = await db.insert(users).values(data).returning();
    return user;
  }

  async update(id: number, data: Partial<NewUser>): Promise<User | undefined> {
    const [user] = await db
      .update(users)
      .set({ ...data, updatedAt: new Date() })
      .where(eq(users.id, id))
      .returning();
    return user;
  }

  async delete(id: number): Promise<boolean> {
    const [deleted] = await db
      .delete(users)
      .where(eq(users.id, id))
      .returning();
    return !!deleted;
  }
}

// Export a singleton instance
export const userRepository = new UserRepository();
```

---

## Express Routes

### `src/routes/users.ts`

```typescript
import { Router, Request, Response } from "express";
import { userRepository } from "../repositories/userRepository";

const router = Router();

// GET /users?page=1&limit=10&search=kavin
router.get("/", async (req: Request, res: Response) => {
  try {
    const page   = Number(req.query.page)   || 1;
    const limit  = Number(req.query.limit)  || 10;
    const search = req.query.search as string | undefined;

    const [data, total] = await Promise.all([
      search
        ? userRepository.search(search)
        : userRepository.findAll(page, limit),
      userRepository.count(),
    ]);

    res.json({ data, total, page, limit });
  } catch (err) {
    res.status(500).json({ error: "Failed to fetch users" });
  }
});

// GET /users/:id
router.get("/:id", async (req: Request, res: Response) => {
  try {
    const user = await userRepository.findById(Number(req.params.id));
    if (!user) return res.status(404).json({ error: "User not found" });
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: "Failed to fetch user" });
  }
});

// POST /users
router.post("/", async (req: Request, res: Response) => {
  try {
    const { name, email, age, role } = req.body;

    // Basic validation
    if (!name || !email) {
      return res.status(400).json({ error: "name and email are required" });
    }

    // Check for duplicate email
    const existing = await userRepository.findByEmail(email);
    if (existing) {
      return res.status(409).json({ error: "Email already in use" });
    }

    const user = await userRepository.create({ name, email, age, role });
    res.status(201).json(user);
  } catch (err) {
    res.status(500).json({ error: "Failed to create user" });
  }
});

// PATCH /users/:id
router.patch("/:id", async (req: Request, res: Response) => {
  try {
    const user = await userRepository.update(Number(req.params.id), req.body);
    if (!user) return res.status(404).json({ error: "User not found" });
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: "Failed to update user" });
  }
});

// DELETE /users/:id
router.delete("/:id", async (req: Request, res: Response) => {
  try {
    const deleted = await userRepository.delete(Number(req.params.id));
    if (!deleted) return res.status(404).json({ error: "User not found" });
    res.status(204).send();
  } catch (err) {
    res.status(500).json({ error: "Failed to delete user" });
  }
});

export default router;
```

### `src/index.ts`

```typescript
import express from "express";
import dotenv from "dotenv";
import usersRouter from "./routes/users";

dotenv.config();

const app  = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// Routes
app.use("/users", usersRouter);

// Health check
app.get("/health", (_req, res) => res.json({ status: "ok" }));

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## Running the Project

```bash
# 1. Clone and install
git clone <your-repo> && cd my-app
npm install

# 2. Set up .env (copy the example above)
cp .env.example .env

# 3. Generate and run migrations
npx drizzle-kit generate
npx drizzle-kit migrate

# 4. Start the development server
npm run dev
```

---

## NPM Scripts

Add these to your `package.json`:

```json
{
  "scripts": {
    "dev":         "tsx watch src/index.ts",
    "build":       "tsc",
    "start":       "node dist/index.js",
    "db:generate": "drizzle-kit generate",
    "db:migrate":  "drizzle-kit migrate",
    "db:push":     "drizzle-kit push",
    "db:studio":   "drizzle-kit studio",
    "db:reset":    "drizzle-kit drop && drizzle-kit push"
  }
}
```

| Script | Purpose |
|---|---|
| `npm run dev` | Start dev server with hot reload |
| `npm run build` | Compile TypeScript to JavaScript |
| `npm start` | Run compiled production build |
| `npm run db:generate` | Generate SQL migration from schema changes |
| `npm run db:migrate` | Apply pending migrations to the DB |
| `npm run db:push` | Push schema directly (dev only) |
| `npm run db:studio` | Open visual DB browser at localhost:4983 |

---

## Testing & Debugging

### Log Generated SQL

Enable Drizzle's built-in query logger during development:

```typescript
export const db = drizzle(pool, {
  schema,
  logger: process.env.NODE_ENV === "development", // logs all SQL to console
});
```

### Use Drizzle Studio

```bash
npm run db:studio
```

Opens a browser UI at `http://localhost:4983` — browse tables, run queries, inspect data.

### Test Queries in Isolation

Create a `src/db/seed.ts` file to test queries directly:

```typescript
import { db } from "./index";
import { users } from "./schema";

async function main() {
  const result = await db.select().from(users).limit(5);
  console.log(result);
  process.exit(0);
}

main().catch(console.error);
```

Run it with: `npx tsx src/db/seed.ts`

### Check Connection

```typescript
import { sql } from "drizzle-orm";

const [{ now }] = await db.execute(sql`SELECT NOW() as now`);
console.log("Connected! DB time:", now);
```

---

## FAQ & Troubleshooting

**Q: `relation "users" does not exist` error**

> You haven't run migrations yet. Run `npm run db:migrate` (or `npm run db:push` in dev).

---

**Q: TypeScript can't find types for my schema**

> Make sure `src/db/schema.ts` is included in your `tsconfig.json` → `"include"` array, and that you're importing from the schema file, not from `drizzle-orm` directly.

---

**Q: Should I use `push` or `generate` + `migrate`?**

> Use `push` during **local development** (fast, no migration files). Use `generate` + `migrate` for **staging/production** so migrations are version-controlled and auditable.

---

**Q: How do I handle `undefined` vs `null` in optional fields?**

> Drizzle uses `null` for nullable DB columns. If a column is `.notNull()` in the schema, TypeScript will enforce that you pass a value on insert.

---

**Q: How do I run raw SQL when Drizzle can't express my query?**

```typescript
import { sql } from "drizzle-orm";

const result = await db.execute(
  sql`SELECT id, name FROM users WHERE name ILIKE ${"%" + query + "%"}`
);
```

---

**Q: Connection pool exhausted in production**

> Tune your `Pool` settings in `src/db/index.ts`. Increase `max` for high-traffic apps, or consider PgBouncer as a connection pooler in front of PostgreSQL.

---

**Q: How do I seed the database?**

> Create `src/db/seed.ts` with your initial data inserts, then run `npx tsx src/db/seed.ts`. You can add it as an npm script: `"db:seed": "tsx src/db/seed.ts"`.

---

## License

MIT
