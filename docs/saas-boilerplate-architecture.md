# SaaS Boilerplate: Finalized Technical Architecture & Implementation Plan

This document captures the **decided technical architecture** for a SaaS product boilerplate using **Next.js**, **Expo**, **Supabase**, and a shared monorepo with TypeScript. It reflects finalized design choices after thorough research and serves as a reference for ongoing development and future onboarding.

---

## 🔧 Stack & Tooling Overview

* **Web App**: [Next.js 15](https://nextjs.org/) with App Router
* **Mobile App**: [Expo](https://expo.dev/) (React Native targeting Android & iOS)
* **Navigation**: [Solito](https://github.com/nandorojo/solito)
* **UI Library**: [Gluestack UI](https://gluestack.io/ui) with NativeWind for styling
* **Backend & Auth**: [Supabase](https://supabase.com/)
* **Animation**: [Moti](https://moti.fyi)
* **Scheduler**: Supabase Cron + `Notifications.scheduleNotificationAsync` for local scheduling
* **Monorepo Tools**: Yarn Workspaces / PNPM + TypeScript Project References
* **Deployment**: Vercel (web), EAS (mobile)
* **E2E Testing**: Playwright (web), Maestro (mobile)

---

## 🏗️ Folder Structure (Monorepo)

```plaintext
/ (root)
├── applications/
│   ├── next/                    # Next.js App Router web app
│   │   └── app/
│   │       ├── (marketing)/    # Public pages (landing, blog)
│   │       ├── (app)/          # Protected routes (dashboard, settings)
│   └── expo/                   # Expo app for iOS/Android
│       └── App.tsx
├── packages/
│   └── app-core/               # Shared modules
│       ├── ui/                 # Gluestack components (React Native + NativeWind)
│       ├── auth/               # Auth workflows and reusable auth UI
│       ├── utils/
│       │   └── database/       # Supabase client setup, RLS rules, migrations
│       ├── config/             # Env, test config, constants
│       └── modules/            # Feature modules (e.g., user/, home/, tasks/)
└── tsconfig.base.json          # TS project references config
```

---

## 🧩 Shared Packages

### 1. `ui/` — Gluestack UI + NativeWind

* Cross-platform UI built using Gluestack
* Styled with NativeWind for consistency across web and native
* Includes atoms (Button, Input) and molecules (Form, Modal, Card)

### 2. `auth/`

* Handles login, registration, logout, session management
* Uses Supabase Auth under the hood
* Exports:

  * Auth context provider
  * React hooks (e.g., `useAuth`, `useSession`)
  * Pre-built auth screens/forms for reuse

### 3. `utils/database/`

* Supabase client setup (including token handling for web/native)
* Includes Supabase `migrations.sql`, RLS policy templates
* Abstracted access layer (`getUserProfile()`, `createTask()`, etc.)

### 4. `config/`

* Exports environment constants loaded via `process.env` (web) and `Constants.manifest.extra` (Expo)
* Shared test configs and build-time flags

### 5. `modules/`

* Each module represents a feature bucket (e.g. `/user`, `/home`, `/tasks`)
* Inside each:

  ```plaintext
  /user
  ├── schema.ts       # Zod schema + DB structure for user
  ├── components/     # Shared UI
  ├── screens/        # Expo screens
  ├── pages/          # Next.js pages
  ├── hooks/          # Business logic hooks
  └── index.ts        # Module entrypoint
  ```
* Encourages domain-driven modularization

---

## 📱 Navigation Strategy — Solito

* Cross-platform routing between Expo and Next.js
* Use `useRouter`, `<Link />`, and shared screen/page definitions
* Layouts and navigation logic abstracted in each platform via Solito adapter

---

## 🎞️ Animation Strategy — Moti

* Use Moti (built on Reanimated) for animations across both web (via RN Web) and native
* Animate shared components (e.g. cards, transitions, modals) inside `ui/`
* Web-only animations (e.g., scroll reveals) can optionally use Framer Motion in Next.js

---

## 🗓️ Scheduling Strategy

* Use **Supabase Cron** for backend-triggered jobs (e.g. reminders, cleanup)
* Local reminders via `Notifications.scheduleNotificationAsync()` in Expo
* Jobs call Supabase Edge Functions or insert records to trigger notifications

---

## 🚀 Deployment & CI/CD

### Web (Next.js)

* Vercel for automatic CI/CD from GitHub
* Route groups for public/protected pages
* Server components for secure auth
* Vercel Cron Jobs for lightweight background tasks (optional)

### Mobile (Expo)

* EAS Build + Submit for iOS/Android binary creation
* EAS Update for OTA updates
* EAS GitHub integration for automated release pipeline
* Versioned releases with semver via `eas.json`

---

## ✅ E2E Testing Setup

### Web

* [Playwright](https://playwright.dev/)
* Test login flow, dashboard interactions, etc.
* GitHub Action for CI

### Mobile

* [Maestro](https://maestro.mobile.dev/) for declarative test flows
* Stored as `.yml` flows in `e2e/`
* CI-compatible, fast and low-flake

---

## 🔐 Security Best Practices

* Supabase RLS policies enforced at DB layer
* No secrets in mobile — use anon keys + secure storage
* Secure headers and CSP in Next.js
* Passwords never handled manually — rely on Supabase Auth

---

## 🧪 TypeScript & Tooling

* Full TS support across all apps and packages
* Project references for scalable builds
* Linting via shared ESLint config
* Paths aliased via `tsconfig.paths`
* Prettier for formatting consistency

---

## 🧭 Git Strategy & Branching Model

We follow a **Git Flow-inspired branching model**, optimized for a solo-to-small team setup:

### 🔄 Branch Types

| Branch      | Purpose                                                                 |
| ----------- | ----------------------------------------------------------------------- |
| `main`      | Always production-ready. Deploys to **production** (Vercel, EAS).       |
| `develop`   | Ongoing development. Deploys to **staging** or preview environments.    |
| `feature/*` | New features, WIP, experimental changes. Branched off `develop`.        |
| `hotfix/*`  | Urgent fixes. Branched off `main`, merged into both `main` + `develop`. |
| `release/*` | Prepares a release. Final testing and versioning before merging.        |

### 🛠️ Workflow Summary

1. **Create a feature**:

   ```bash
   git checkout develop
   git checkout -b feature/feature-name
   ```
2. **Finish a feature**:

   * Ensure tests pass
   * Open a PR into `develop`
3. **Create a release**:

   ```bash
   git checkout develop
   git checkout -b release/vX.X.X
   ```

   * Final test & version bump
   * Merge to `main` and `develop`
4. **Deployments**:

   * `main`: triggers **production** deploy (web & mobile builds)
   * `develop`: optional deploy to **staging**
   * Tag `main` (e.g., `v1.2.0`) to trigger **EAS build** for mobile
5. **Hotfixes**:

   ```bash
   git checkout main
   git checkout -b hotfix/fix-issue
   ```

### 📑 Commit Message Conventions (Optional)

Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for clarity and potential changelog automation:

```
feat: add user profile card
fix: resolve login form validation issue
chore: update dependencies
```

---

## 📘 Developer Onboarding Checklist

1. Install dependencies via `pnpm i`
2. Run dev apps:

   * `pnpm dev:web` for Next.js
   * `pnpm dev:mobile` for Expo
3. Setup Supabase credentials in `.env`
4. Run E2E tests via `pnpm test:e2e:web` or `pnpm test:e2e:mobile`

---

## 📌 Final Notes

This setup is optimized for **solopreneurs** and **small teams** aiming for maximum reuse, clean architecture, and modern UX. It enables smooth cross-platform development with scalable backend integrations and efficient CI/CD pipelines. All technical decisions are documented and modularized for future growth.

---

For architectural rationale, refer to the companion document: `architecture-options-and-research.md`
