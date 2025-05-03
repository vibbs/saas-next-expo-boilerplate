# 🗄️ Prompt: SQL Migration Script

**Context**
SQL lives in `packages/app-core/utils/database/migrations/`.
We use Postgres + Supabase.

**Task**
Given this schema change description: `<description>`, write an **idempotent** migration.

**Requirements**
1. Use `IF NOT EXISTS` / `IF EXISTS`.
2. Include RLS policy stubs where applicable.
3. End with `COMMIT;`.

**Output**
Return a single fenced `sql` block; file name comment on first line.
