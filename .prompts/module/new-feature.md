# 🏗️ Prompt: Scaffold a New Feature Module

**Context**  
Feature modules live in `packages/app-core/modules/<feature>/` as described in `.clinerules/structure.md`.

---

## Task
Generate the folder and starter files for a feature named **`<feature>`**.

### Required files & purpose
| Path | Description |
|------|-------------|
| `packages/app-core/modules/<feature>/schema.ts` | Zod schema + Supabase‐generated types |
| `packages/app-core/modules/<feature>/screens/<Feature>Screen.tsx` | Expo screen component |
| `packages/app-core/modules/<feature>/pages/<feature>/page.tsx` | Next.js App-Router page component |
| `packages/app-core/modules/<feature>/hooks/use<Feature>.ts` | Business-logic React hook |
| `packages/app-core/modules/<feature>/index.ts` | Barrel export |
| `packages/app-core/modules/<feature>/README.md` | Short module overview & TODO list |

---

## Constraints
1. All code must be **TypeScript** and follow rules in `.clinerules/code-standards.md`.
2. UI uses **Gluestack UI** + **NativeWind** utilities.
3. Screen/Page components should import shared UI primitives, not duplicate styles.
4. Placeholder components must compile and render “Work in progress…”.
5. Add inline TODO comments indicating where real logic will go.

---

## Output
Create the files in respective designated paths with content, for example:

```tsx title=packages/app-core/modules/<feature>/screens/<Feature>Screen.tsx
// packages/app-core/modules/example/screens/ExampleScreen.tsx
```