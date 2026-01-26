---
name: postgres-best-practices
description: Postgres and Supabase best practices for AI agents. Use when writing database queries, schema changes, RLS policies, or optimizing database performance. Based on Supabase's 30 rules for production-ready Postgres code.
---

# /postgres-best-practices - Database Best Practices

## When to Use

Invoke `/postgres-best-practices` when:
- Writing or modifying database queries
- Creating or altering schema/migrations
- Implementing Row Level Security (RLS) policies
- Optimizing database performance
- Debugging slow queries
- Working with multi-tenant data

## Priority Categories

| Priority | Category | Impact |
|----------|----------|--------|
| 1 | Query Performance | CRITICAL |
| 2 | Connection Management | CRITICAL |
| 3 | Security & RLS | CRITICAL |
| 4 | Schema Design | HIGH |
| 5 | Concurrency & Locking | MEDIUM-HIGH |
| 6 | Data Access Patterns | MEDIUM |
| 7 | Monitoring & Diagnostics | LOW-MEDIUM |
| 8 | Advanced Features | LOW |

---

## 1. Query Performance (CRITICAL)

### 1.1 Always Use Indexes for Filtered Columns

**BAD - Full table scan:**
```sql
SELECT * FROM leads WHERE organization_id = 'org_123';
```

**GOOD - With index:**
```sql
-- Create index first
CREATE INDEX idx_leads_organization_id ON leads(organization_id);

-- Query now uses index
SELECT * FROM leads WHERE organization_id = 'org_123';
```

**In Drizzle migrations:**
```typescript
export async function up(db: PostgresJsDatabase) {
  await db.execute(sql`
    CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_leads_organization_id
    ON leads(organization_id)
  `);
}
```

### 1.2 Use Composite Indexes for Multi-Column Filters

**Common multi-tenant pattern:**
```sql
-- For queries filtering by org + status
CREATE INDEX idx_leads_org_status ON leads(organization_id, status);

-- For queries filtering by org + date range
CREATE INDEX idx_bookings_org_date ON bookings(organization_id, created_at DESC);
```

**Order matters:** Put equality columns first, range columns last.

### 1.3 Avoid SELECT *

**BAD:**
```typescript
const leads = await db.select().from(leads).where(eq(leads.organizationId, orgId));
```

**GOOD - Select only needed columns:**
```typescript
const leads = await db
  .select({
    id: leads.id,
    name: leads.name,
    email: leads.email,
    status: leads.status,
  })
  .from(leads)
  .where(eq(leads.organizationId, orgId));
```

### 1.4 Use EXPLAIN ANALYZE for Slow Queries

```sql
EXPLAIN ANALYZE SELECT * FROM messages WHERE chat_id = 'chat_123';
```

Look for:
- `Seq Scan` → Missing index
- High `actual time` → Slow query
- High `rows` → Too much data returned

### 1.5 Avoid N+1 Queries

**BAD - N+1 pattern:**
```typescript
const chats = await db.select().from(chats);
for (const chat of chats) {
  const messages = await db.select().from(messages).where(eq(messages.chatId, chat.id));
}
```

**GOOD - Single query with join:**
```typescript
const chatsWithMessages = await db
  .select()
  .from(chats)
  .leftJoin(messages, eq(chats.id, messages.chatId))
  .where(eq(chats.organizationId, orgId));
```

### 1.6 Use Partial Indexes for Common Filters

```sql
-- Index only active leads (smaller, faster)
CREATE INDEX idx_leads_active ON leads(organization_id)
WHERE status = 'active';

-- Index only unread messages
CREATE INDEX idx_messages_unread ON messages(chat_id)
WHERE is_read = false;
```

---

## 2. Connection Management (CRITICAL)

### 2.1 Use Connection Pooling

**Using Supabase connection pooler:**
```typescript
// Use pooler URL for serverless
const connectionString = process.env.DATABASE_URL; // Uses Supavisor pooler

// For direct access (migrations only)
const directUrl = process.env.DIRECT_URL;
```

### 2.2 Close Connections in Serverless

**BAD - Connection leak:**
```typescript
export async function GET() {
  const db = drizzle(postgres(process.env.DATABASE_URL));
  const data = await db.select().from(users);
  return Response.json(data);
  // Connection never closed!
}
```

**GOOD - Use shared client:**
```typescript
// lib/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';

const client = postgres(process.env.DATABASE_URL!);
export const db = drizzle(client);

// API route
import { db } from '@/lib/db';

export async function GET() {
  const data = await db.select().from(users);
  return Response.json(data);
}
```

### 2.3 Set Statement Timeout

```typescript
const client = postgres(process.env.DATABASE_URL!, {
  max: 10, // Connection pool size
  idle_timeout: 20,
  connect_timeout: 10,
  statement_timeout: 30000, // 30s max query time
});
```

---

## 3. Security & RLS (CRITICAL)

### 3.1 ALWAYS Enable RLS for Multi-Tenant Tables

**Every tenant-specific table MUST have RLS:**
```sql
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;
ALTER TABLE leads FORCE ROW LEVEL SECURITY; -- Prevents owner bypass
```

### 3.2 Create Organization-Based Policies

**Standard multi-tenant pattern:**
```sql
-- Users can only see their organization's data
CREATE POLICY "Users can view own org leads"
ON leads FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
);

-- Users can only insert into their organization
CREATE POLICY "Users can insert own org leads"
ON leads FOR INSERT
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
);
```

### 3.3 NEVER Rely on Application-Level Filtering Alone

**BAD - Vulnerable if filter bypassed:**
```typescript
// Application-only filtering
const leads = await db
  .select()
  .from(leads)
  .where(eq(leads.organizationId, userOrgId)); // What if userOrgId is spoofed?
```

**GOOD - Database enforces security:**
```sql
-- RLS policy ensures only authorized data returned
-- Even if application bug exists, database protects data
```

### 3.4 Use Service Role Carefully

**Service role bypasses RLS. Only use for:**
- Admin operations
- Background jobs
- System migrations

```typescript
// Regular client (RLS enforced)
import { createClient } from '@supabase/supabase-js';
const supabase = createClient(url, anonKey);

// Service role (RLS bypassed - use carefully)
const supabaseAdmin = createClient(url, serviceRoleKey);
```

### 3.5 Validate Organization Access in API Routes

**Always verify organization membership:**
```typescript
export async function GET(req: Request) {
  const { userId } = await auth();
  const organizationId = req.headers.get('x-organization-id');

  // Verify user belongs to organization
  const membership = await db
    .select()
    .from(organizationMembers)
    .where(
      and(
        eq(organizationMembers.userId, userId),
        eq(organizationMembers.organizationId, organizationId)
      )
    )
    .limit(1);

  if (!membership.length) {
    return new Response('Unauthorized', { status: 403 });
  }

  // Now safe to query organization data
}
```

---

## 4. Schema Design (HIGH)

### 4.1 Use UUIDs for Primary Keys

```typescript
// schema.ts
export const leads = pgTable('leads', {
  id: uuid('id').primaryKey().defaultRandom(),
  // ...
});
```

### 4.2 Always Include Organization ID

**Every tenant-specific table needs:**
```typescript
export const leads = pgTable('leads', {
  id: uuid('id').primaryKey().defaultRandom(),
  organizationId: uuid('organization_id')
    .notNull()
    .references(() => organizations.id, { onDelete: 'cascade' }),
  // ... other columns
});
```

### 4.3 Use Proper Data Types

| Data | Type | Example |
|------|------|---------|
| Money | `numeric(10,2)` | `price numeric(10,2)` |
| Timestamps | `timestamptz` | `created_at timestamptz` |
| JSON | `jsonb` | `metadata jsonb` |
| Enums | `pgEnum` | `status lead_status` |
| Arrays | `text[]` | `tags text[]` |

**Drizzle example:**
```typescript
export const leadStatusEnum = pgEnum('lead_status', [
  'new', 'contacted', 'qualified', 'converted', 'lost'
]);

export const leads = pgTable('leads', {
  status: leadStatusEnum('status').default('new'),
  metadata: jsonb('metadata').$type<LeadMetadata>(),
  tags: text('tags').array(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
});
```

### 4.4 Add Soft Delete for Important Data

```typescript
export const leads = pgTable('leads', {
  // ...
  deletedAt: timestamp('deleted_at', { withTimezone: true }),
});

// Query with soft delete filter
const activeLeads = await db
  .select()
  .from(leads)
  .where(and(
    eq(leads.organizationId, orgId),
    isNull(leads.deletedAt)
  ));
```

### 4.5 Use Foreign Key Constraints

```typescript
export const messages = pgTable('messages', {
  chatId: uuid('chat_id')
    .notNull()
    .references(() => chats.id, { onDelete: 'cascade' }),
  userId: uuid('user_id')
    .references(() => users.id, { onDelete: 'set null' }),
});
```

---

## 5. Concurrency & Locking (MEDIUM-HIGH)

### 5.1 Use SKIP LOCKED for Job Queues

```sql
-- Fetch next job without blocking
SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY created_at
LIMIT 1
FOR UPDATE SKIP LOCKED;
```

### 5.2 Avoid Long Transactions

**BAD - Holds locks too long:**
```typescript
await db.transaction(async (tx) => {
  const lead = await tx.select().from(leads).where(...);
  await someExternalApi(); // Long-running operation!
  await tx.update(leads).set({ status: 'processed' }).where(...);
});
```

**GOOD - Minimize transaction scope:**
```typescript
const lead = await db.select().from(leads).where(...);
const result = await someExternalApi(); // Outside transaction

await db.transaction(async (tx) => {
  await tx.update(leads).set({ status: 'processed' }).where(...);
});
```

### 5.3 Use Optimistic Locking for Concurrent Updates

```typescript
export const leads = pgTable('leads', {
  // ...
  version: integer('version').default(1),
});

// Update with version check
const result = await db
  .update(leads)
  .set({
    name: newName,
    version: sql`${leads.version} + 1`
  })
  .where(and(
    eq(leads.id, leadId),
    eq(leads.version, currentVersion)
  ));

if (result.rowCount === 0) {
  throw new Error('Concurrent modification detected');
}
```

---

## 6. Data Access Patterns (MEDIUM)

### 6.1 Use Cursor-Based Pagination

**BAD - OFFSET is slow for large datasets:**
```typescript
const page10 = await db
  .select()
  .from(leads)
  .limit(20)
  .offset(180); // Scans 200 rows to skip 180
```

**GOOD - Cursor-based:**
```typescript
const nextPage = await db
  .select()
  .from(leads)
  .where(gt(leads.createdAt, lastCursor))
  .orderBy(desc(leads.createdAt))
  .limit(20);
```

### 6.2 Batch Inserts

**BAD - Individual inserts:**
```typescript
for (const lead of leads) {
  await db.insert(leadsTable).values(lead);
}
```

**GOOD - Batch insert:**
```typescript
await db.insert(leadsTable).values(leads); // Single query
```

### 6.3 Use Upsert for Idempotency

```typescript
await db
  .insert(leads)
  .values({ id: leadId, email, name, organizationId })
  .onConflictDoUpdate({
    target: leads.id,
    set: { email, name, updatedAt: new Date() },
  });
```

---

## 7. Monitoring & Diagnostics (LOW-MEDIUM)

### 7.1 Log Slow Queries

```sql
-- In Supabase dashboard: Database → Settings → Log min duration statement
-- Set to 1000ms to log queries > 1 second
```

### 7.2 Check Index Usage

```sql
SELECT
  schemaname, tablename, indexname,
  idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0  -- Unused indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

### 7.3 Find Missing Indexes

```sql
SELECT
  relname as table,
  seq_scan,
  idx_scan,
  seq_tup_read,
  idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan  -- More sequential scans than index scans
ORDER BY seq_tup_read DESC;
```

---

## 8. Advanced Features (LOW)

### 8.1 Use CTEs for Complex Queries

```typescript
const leadsWithStats = await db.execute(sql`
  WITH lead_stats AS (
    SELECT
      lead_id,
      COUNT(*) as message_count,
      MAX(created_at) as last_activity
    FROM lead_activities
    WHERE organization_id = ${orgId}
    GROUP BY lead_id
  )
  SELECT l.*, s.message_count, s.last_activity
  FROM leads l
  LEFT JOIN lead_stats s ON l.id = s.lead_id
  WHERE l.organization_id = ${orgId}
`);
```

### 8.2 Use Window Functions for Analytics

```sql
SELECT
  date_trunc('day', created_at) as day,
  COUNT(*) as leads_count,
  SUM(COUNT(*)) OVER (ORDER BY date_trunc('day', created_at)) as cumulative
FROM leads
WHERE organization_id = 'org_123'
GROUP BY date_trunc('day', created_at);
```

### 8.3 Use JSONB for Flexible Metadata

```typescript
// Query JSONB fields
const leads = await db
  .select()
  .from(leads)
  .where(sql`metadata->>'source' = 'facebook'`);

// Update JSONB fields
await db
  .update(leads)
  .set({
    metadata: sql`metadata || '{"qualified": true}'::jsonb`
  })
  .where(eq(leads.id, leadId));
```

---

## Common Application Patterns

### Multi-Tenant Query Pattern

```typescript
// Always filter by organizationId
async function getLeads(organizationId: string) {
  return db
    .select()
    .from(leads)
    .where(eq(leads.organizationId, organizationId))
    .orderBy(desc(leads.createdAt));
}
```

### Chat Messages with Pagination

```typescript
async function getMessages(chatId: string, cursor?: Date) {
  return db
    .select()
    .from(messages)
    .where(and(
      eq(messages.chatId, chatId),
      cursor ? lt(messages.createdAt, cursor) : undefined
    ))
    .orderBy(desc(messages.createdAt))
    .limit(50);
}
```

### Subscription Limit Checking

```typescript
async function checkLimit(organizationId: string, limitType: string) {
  const [result] = await db
    .select({
      currentUsage: limits.currentUsage,
      maxLimit: limits.maxLimit,
    })
    .from(limits)
    .where(and(
      eq(limits.organizationId, organizationId),
      eq(limits.limitType, limitType)
    ))
    .limit(1);

  return result.currentUsage < result.maxLimit;
}
```

---

## Pre-Query Checklist

Before writing any database query:

- [ ] Is there an index for the WHERE columns?
- [ ] Am I filtering by organizationId (multi-tenant)?
- [ ] Am I selecting only needed columns (not SELECT *)?
- [ ] Is RLS enabled on the table?
- [ ] Am I using transactions correctly (minimal scope)?
- [ ] Is this query paginated (for list endpoints)?

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/react-best-practices` | Frontend performance optimization |
| `/plan` | Planning database schema changes |
| `/implement` | Implementing database queries |
