# 🧩 Prompt: Generate a Cross-Platform UI Component

**Context**
- Project rules live in `.clinerules/structure.md` & `.clinerules/code-standards.md`.
- Components reside in **packages/app-core/ui** and must be compatible with React Native Web.

**Task**
Create a new component named `<ComponentName>`.

**Requirements**
1. Written in **TypeScript** and uses **Gluestack UI** + **NativeWind** utilities only.
2. Accepts props typed with an exported interface.
3. No direct DOM or Platform imports; use RN primitives.
4. Add a `*.stories.tsx` Storybook file and `*.test.tsx` (Vitest + RTL).
5. Follow accessibility best practices (`accessibilityRole`, `aria-*` on web).
6. Follow naming & file-placement rules.

**Output**  
Create the files in respective designated paths with right content.
Create three files as mentioned below

```tsx title=Component
// path: packages/app-core/ui/<ComponentName>/<ComponentName>.tsx
```

```tsx title=Test
// path: packages/app-core/ui/<ComponentName>/<ComponentName>.test.tsx

```

```tsx title=Story
// path: packages/app-core/ui/<ComponentName>/<ComponentName>.stories.tsx

```
