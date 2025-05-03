# ✨ Code Standards

## 1. TypeScript

* Strict mode on (`noImplicitAny`, `strictNullChecks`).
* Zero `any`; use `unknown` + type-guards.
* Generate DB types via `supabase gen types`.  
  Never hand-craft table interfaces.

## 2. React / Next.js / Expo

| Rule | Notes |
|------|-------|
| Prefer **Server Components** in Next | Mark with `/* @server */` JSDoc |
| Expo screens are **Client Components** | Use React Navigation hooks |
| All shared UI lives in `packages/app-core/ui` | Built with **Gluestack UI** + **NativeWind** |
| Animations via **Moti** only | No inline `Animated` API unless justified in PR |

## 3. Styling

* Tailwind config is the single source of design tokens.  
  NativeWind consumes the same config for RN.
* Do **not** write custom CSS outside Tailwind utilities except inside `global.css` or component-scoped classes.

## 4. Error Handling

* Wrap async calls with `try/catch`; surface errors via toast or error boundary.
* Do not log secrets; redact tokens before console/logging.

AI agents must run `pnpm lint && pnpm test` before opening a PR.
