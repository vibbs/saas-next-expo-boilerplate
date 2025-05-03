# 🌳 Git Workflow

We follow **Git Flow** with Conventional Commits.

## 1. Branches

* `main` – production releases (auto-deploy Vercel & EAS)
* `develop` – integration / staging
* `feature/<slug>` – new work from `develop`
* `hotfix/<slug>` – urgent patch from `main`
* `release/vX.Y.Z` – prep staging → prod

## 2. Commits

```txt
feat(auth): add passwordless login
fix(ui): button hover state
```

A bot rejects non-conforming messages.


## 3. Pull Requests
Title == first commit message.

Checklist must pass: lint, type-check, tests, Percy/Chromatic.

Link issue or story ID (P3-S3 etc.).

One reviewer minimum (self-review not allowed).

## 4. Tags & Releases
Tag main with vX.Y.Z to trigger EAS build.

semantic-release writes release notes to GitHub.

```

---

## 📁 `.clinerules/testing.md`

```md
# 🧪 Testing Rules

| Layer | Tool | Min Coverage |
|-------|------|-------------|
| Unit  | Vitest + RTL        | 80 % lines |
| UI    | Storybook + Chromatic| N/A (visual diff) |
| E2E Web | Playwright        | Smoke flows |
| E2E Mobile | Maestro        | Auth flows |

## General

1. Write tests **before** or with implementation (TDD recommended for AI bots).
2. Use `data-testid` only when semantic selectors are impossible.
3. Mock network with **MSW**; never hit real APIs in unit tests.

```