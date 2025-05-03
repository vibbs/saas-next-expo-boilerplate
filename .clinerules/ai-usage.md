# 🤖 AI Tools Usage Rules

These rules apply to *any* automated code-generation tool (Copilot, Cursor, Codeium, GPT CLI, etc.).

---

## 1. Allowed Scenarios

| Scenario | Examples |
|----------|----------|
| **Boilerplate** | Generate repetitive files (e.g. Storybook stories, Jest test shells, Tailwind variants). |
| **Bulk refactor** | Rename a prop across all modules, migrate imports, convert CommonJS → ESM. |
| **Doc scaffolding** | Create initial ADR, README skeletons, JSDoc blocks. |
| **SQL migration** | Produce `CREATE TABLE` / `ALTER TABLE` scripts from a written schema spec. |

> If a task is *creative* or *business-critical* (e.g. auth logic, payment flow), a human must **pair review** the AI output line-by-line before commit.

---

## 2. Mandatory Pre-commit Checklist

Every AI-generated change **must** pass:

1. `pnpm lint`, `pnpm prettier:check`, `pnpm test` – zero errors.
2. Project structure rules in [`structure.md`](./structure.md) (paths, naming).
3. Code-style rules in [`code-standards.md`](./code-standards.md).
4. If new files are added, **matching tests** or stories are added as well.

The CI pipeline will block the PR if any of the above fail.

---

## 3. Prompt Library (Optional)

To keep prompts consistent, store reusable prompt files in:

```
.prompts/<topic>/<file>.md
```

Example file: `.prompts/ui/new-component.md`

```md
You are working inside **packages/app-core/ui**.
Goal: create <ComponentName> that follows project rules (see .clinerules).
Requirements:
- TypeScript + React Native Web compatible
- Tailwind classes only
Return **one** code block per file: component, test, story.
```

## 4. Commit Footer for AI Authorship
Add a trailer line so we can audit machine contributions:

```
Co-authored-by: AI Assistant <bot@univas.com>
```


## 5. Escalation & Review
🚧 Red-flag areas (auth, payments, encryption) require human pair review even if tests pass.

If the AI output introduces a new dependency, open an ADR (docs/adr/) before merging.

```

---

### Key take-aways

* The file is **tool-agnostic**—just uses a neutral `.prompts/` folder.
* It spells out exactly **when** AI can generate code and the **minimum gates** it must clear.
* The “commit footer” is optional but keeps an easily searchable audit trail.

Let me know if you’d like further tweaks or examples for a specific AI workflow!

```