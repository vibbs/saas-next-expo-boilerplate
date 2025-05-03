# 📦 Project Structure Rules

> **Scope:** Applies to humans & AI tools that add/move files.

## 1. Root Layout

```

/
├── applications/ # Runtime targets
│ ├── next/ # Next 15 App-Router app
│ └── expo/ # Expo React-Native app
├── packages/
│ └── app-core/
│ ├── ui/ # Gluestack + NativeWind components
│ ├── auth/ # Cross-platform auth flows
│ ├── utils/
│ │ └── database/ # Supabase client, SQL, cron
│ ├── config/ # Env, build flags
│ └── modules/ # Feature domains (one folder each)
├── .clinerules/ # <-- you are here
└── turbo.json # Task graph
```

*Never create new top-level folders without an ADR* (`docs/adr/`).

## 2. Feature Module Layout

Every domain lives in `packages/app-core/modules/<feature>/` :


```
/<feature>
├── schema.ts # Zod + DB types
├── components/ # Pure UI
├── screens/ # Expo screens
├── pages/ # Next pages (App Router)
├── hooks/ # React logic
└── index.ts # Barrel export
```



## 3. Naming

* kebab-case for folders, `PascalCase.tsx` for React components.  
* Tests co-locate as `*.test.ts(x)`.  
* Stories go in `*.stories.tsx`.

Violations must be flagged in PR review and corrected before merge.
