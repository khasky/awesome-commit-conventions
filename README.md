# Conventional Commits + SemVer Cheatsheet

A practical, no-fluff cheatsheet for writing clean commit messages with **[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)** and versioning releases with **[Semantic Versioning](https://semver.org/)**. Examples use the Node.js / JavaScript ecosystem (`npm`, `commitlint`, `semantic-release`).

---

## TL;DR

```
<type>(<scope>): <short summary>
<BLANK LINE>
<body: why the change, what was wrong>
<BLANK LINE>
<footer: issue refs, BREAKING CHANGE, Co-authored-by>
```

- Only the **first line is required**. Body and footer are optional.
- Write the summary in the **imperative mood**: `add`, `fix`, `remove` — *not* `added` / `adds` / `fixing`.
  - Mnemonic: *"If applied, this commit will → **add search endpoint**."*
- The spec mandates **only two types**: `feat` and `fix`. Everything else is convention.
- `feat` → **minor** bump, `fix` → **patch** bump, `!` or `BREAKING CHANGE:` → **major** bump.

---

## Table of Contents

- [Conventional Commits + SemVer Cheatsheet](#conventional-commits--semver-cheatsheet)
  - [TL;DR](#tldr)
  - [Table of Contents](#table-of-contents)
  - [Anatomy of a commit](#anatomy-of-a-commit)
  - [The first line (the only required part)](#the-first-line-the-only-required-part)
  - [Commit types](#commit-types)
    - [What exactly is `chore`?](#what-exactly-is-chore)
  - [Scopes](#scopes)
  - [Body \& footer: when to add them](#body--footer-when-to-add-them)
  - [Breaking changes](#breaking-changes)
  - [A realistic history (20 commits)](#a-realistic-history-20-commits)
  - [Examples by area](#examples-by-area)
  - [Semantic Versioning (SemVer)](#semantic-versioning-semver)
  - [How Conventional Commits drive SemVer](#how-conventional-commits-drive-semver)
  - [Node.js tooling](#nodejs-tooling)
    - [1. Lint commit messages — `commitlint` + `husky`](#1-lint-commit-messages--commitlint--husky)
    - [2. Guided prompts (optional) — `commitizen`](#2-guided-prompts-optional--commitizen)
    - [3. Automated releases — pick one](#3-automated-releases--pick-one)
    - [4. Plain npm (no extra tooling)](#4-plain-npm-no-extra-tooling)
  - [The first commit](#the-first-commit)
  - [Myth-busting: what the spec actually requires](#myth-busting-what-the-spec-actually-requires)
  - [Reality check: what big projects really do](#reality-check-what-big-projects-really-do)
  - [Other conventions](#other-conventions)
  - [What should *you* use?](#what-should-you-use)
  - [One-page cheat card](#one-page-cheat-card)
  - [Sources](#sources)
  - [License](#license)

---

## Anatomy of a commit

```
feat(search): add full-text search endpoint
│    │         │
│    │         └─⫸ Summary: imperative, ≤ ~50 chars, no trailing period
│    └─────────⫸ Scope (optional): the module/area touched
└──────────────⫸ Type: feat, fix, docs, refactor, perf, test, build, ci, chore, style, revert
```

Full form with body and footer:

```
fix(auth): reject tokens issued before password reset

Previously a leaked token stayed valid after the user reset their
password. Now we compare the token's `iat` against the last
password-change timestamp.

Closes #482
```

The **header** answers *what*. The **body** answers *why*. The **footer** carries metadata (issues, breaking changes, co-authors).

---

## The first line (the only required part)

| Rule | Do | Don't |
|---|---|---|
| Mood | `add search endpoint` | `added` / `adds` / `adding` |
| Length | ≤ ~50 characters | a paragraph in the header |
| Casing | `feat: add ...` (lowercase after `:`) | `Feat: Add ...` (see [lowercase reality](#myth-busting-what-the-spec-actually-requires)) |
| Punctuation | no period at the end | `feat: add search.` |
| Scope | optional, in parentheses | inconsistent names for the same module |

> **The 50/72 rule** (popularized by the Linux kernel team long before Angular): keep the
> **summary ≤ 50 chars** and **wrap body lines at 72 chars**. It's the most universal
> discipline in git — even outside Conventional Commits.

---

## Commit types

Only `feat` and `fix` are defined by the spec. The rest come from the **Angular convention** and are recommended by the `commitlint` preset:

| Type | When to use | SemVer impact |
|---|---|---|
| `feat` | A new feature | **minor** (`x.Y.z`) |
| `fix` | A bug fix | **patch** (`x.y.Z`) |
| `docs` | Documentation only | none |
| `refactor` | Code change that neither fixes a bug nor adds a feature | none |
| `perf` | Performance improvement | none (or patch) |
| `test` | Adding or fixing tests | none |
| `build` | Build system or dependencies (`npm`, bundler) | none |
| `ci` | CI configuration and pipelines | none |
| `chore` | Routine maintenance, not user-facing | none |
| `style` | Formatting, whitespace — no logic change | none |
| `revert` | Reverts a previous commit | depends |

Add `!` after the type/scope for a **breaking change** — it bumps the **major** version regardless of the type. See [Breaking changes](#breaking-changes).

### What exactly is `chore`?

`chore` is the **catch-all bin for "this doesn't affect users"**: dependency bumps, config tweaks (`.gitignore`, linter settings), version bumps, release publishing. These commits usually **don't appear in the changelog and don't move the version** — the release-notes generator ignores them.

The common mistake: `chore` ≠ "any small change".

- Rewrote code without changing behavior? → `refactor`
- Made it faster? → `perf`
- Fixed formatting? → `style`

`chore` is strictly for git-adjacent housekeeping that doesn't touch feature source code.

---

## Scopes

The scope is **your responsibility area in the project, not a fixed dictionary**. You invent `auth`, `payments`, `ui`, etc. to match your repo structure.

**Rules of thumb:**

- **Be consistent.** If you start with `auth`, never call the same area `authentication` or `login` later — it breaks changelog grouping and `git log --grep` filtering.
- **Type and scope are independent.** Any type pairs with any scope: `feat(auth)`, `fix(auth)`, `perf(auth)`, `test(auth)`.
- **Some types rarely need a scope.** `build`, `ci`, `chore(release)`, and project-wide `docs` are global — parentheses are often noise.
- **Match your folders.** If game modes live in `modes/`, name the scope `modes`.

**Casing — use `kebab-case`:**

```
fix(game-modes): correct scoring in survival mode
feat(game-modes): add ranked matchmaking
```

> The spec technically tolerates spaces (`fix(game modes): ...`), but the default
> `@commitlint/config-conventional` preset enforces `scope-case: lower-case`, so a
> spaced scope will be **rejected** by CI. `kebab-case` is what humans and tools expect.
> `camelCase` and `snake_case` exist but are rarer.

---

## Body & footer: when to add them

**Add a body** when the header alone doesn't explain *why*. If the fix is a one-liner and obvious, skip it. If you changed behavior or made a non-obvious decision, write it down — imagine someone doing `git blame` on your line two years from now.

**The footer** matters in three cases:

**1. Linking an issue** — GitHub auto-closes the issue on merge to the default branch:

```
Closes #482      # also: Fixes #482, Resolves #482
Refs #482        # mention without closing
```

**2. Breaking change** — see the next section.

**3. Co-authorship** — when pairing or applying someone else's patch:

```
Co-authored-by: Jane Doe <jane@example.com>
```

---

## Breaking changes

Two equivalent ways to flag a breaking change. Either triggers a **major** version bump.

**A) `!` in the header** (compact):

```
feat(api)!: change default sort to relevance
```

**B) `BREAKING CHANGE:` in the footer** (explicit, lands in the changelog with a callout):

```
feat(api)!: remove deprecated v1 endpoints

BREAKING CHANGE: /api/v1/* removed. Migrate to /api/v2/*.
```

> `BREAKING CHANGE` is the **one token the spec requires in uppercase**. Everything
> else is case-insensitive to tools.

---

## A realistic history (20 commits)

A feature (adding search) with bug fixes, review churn, and a release:

```
feat(search): add basic full-text search endpoint
feat(search): add pagination support to results
test(search): add unit tests for query parser
fix(search): handle empty query string gracefully
docs(search): document search API in README
refactor(search): extract query builder into separate module
perf(search): add index on title column
fix(search): escape special characters in user input
style: format search module with prettier
test(search): add integration tests for pagination
chore(deps): bump elasticsearch client to 8.2.0
ci: run search tests on pull requests
fix(search): correct off-by-one error in page offset
docs: add search examples to contributing guide
refactor(search): simplify result mapping logic
feat(search): add filter by category
fix(search): return 400 on invalid filter values
revert: "perf(search): add index on title column"
feat(search)!: change default sort to relevance
chore(release): bump version to 1.4.0
```

Two details worth noting:

- A `revert:` commit **repeats the subject** of the commit it undoes.
- `feat(search)!` — the `!` before the colon flags a breaking change **right in the header**.

---

## Examples by area

Same type set, applied across different parts of a project.

**Authentication (`auth`)**
```
feat(auth): add login with email and password
feat(auth): add password reset via email link
fix(auth): reject tokens issued before password change
perf(auth): cache JWT public keys to avoid refetch
test(auth): add tests for refresh token rotation
feat(auth)!: drop support for legacy session cookies
```

**Payments (`payments`)**
```
feat(payments): integrate Stripe checkout
fix(payments): handle webhook retries idempotently
refactor(payments): extract invoice builder into service
docs(payments): document refund flow in wiki
fix(payments): round currency amounts to avoid drift
```

**API (`api`)**
```
feat(api): add v2 endpoints for bulk operations
fix(api): return 429 with Retry-After on rate limit
docs(api): add OpenAPI spec for orders resource
refactor(api): unify error response shape
feat(api)!: remove deprecated v1 endpoints
```

**Database / migrations (`db`)**
```
feat(db): add index on orders.created_at
fix(db): correct foreign key on order_items
perf(db): batch insert for import jobs
refactor(db): rename ambiguous status column
```

**UI (`ui`)**
```
feat(ui): add dark mode toggle
fix(ui): keep dropdown open on keyboard navigation
style(ui): switch to design-token spacing scale
perf(ui): memoize heavy list components
a11y(ui): add ARIA labels to icon buttons
```
> `a11y` is a non-standard but common type for accessibility. If your linter doesn't know it,
> add it to the config or use `fix(ui):` instead.

**Localization (`i18n`)**
```
feat(i18n): add Ukrainian and Polish locales
fix(i18n): correct plural rules for Russian
chore(i18n): sync translation keys with source
```

**Infrastructure & maintenance (no business logic)**
```
build: migrate from webpack to vite
ci: cache node_modules between pipeline runs
chore(deps): bump axios to 1.7.0
chore(deps-dev): update eslint to v9
docs: rewrite installation section in README
revert: "feat(api): add v2 endpoints for bulk operations"
chore(release): v2.0.0
```

---

## Semantic Versioning (SemVer)

A version is **`MAJOR.MINOR.PATCH`** (e.g. `2.4.1`). Given a change, increment:

| Part | Bump when | Example |
|---|---|---|
| **MAJOR** | You make **incompatible** API changes | `1.4.2` → `2.0.0` |
| **MINOR** | You add functionality in a **backward-compatible** way | `1.4.2` → `1.5.0` |
| **PATCH** | You make backward-compatible **bug fixes** | `1.4.2` → `1.4.3` |

**Rules that trip people up:**

- When you bump a higher part, **lower parts reset to 0**: `1.4.2` → `2.0.0`, not `2.4.2`.
- **`0.y.z` is the wild west.** Anything may change at any time; the public API is not stable. Many npm packages sit at `0.x` deliberately.
- **`1.0.0` defines your public API.** After it, you're on the hook for compatibility.
- Once released, **a version is immutable** — never re-publish `1.4.2` with different content; ship `1.4.3`.

**Pre-release and build metadata:**

```
1.0.0-alpha        1.0.0-alpha.1        1.0.0-beta.2        1.0.0-rc.1
1.0.0+20130313144700        1.0.0-beta+exp.sha.5114f85
```

- **Pre-release** (`-alpha`, `-beta.2`, `-rc.1`) has **lower** precedence than the normal version: `1.0.0-rc.1` < `1.0.0`.
- **Build metadata** (`+...`) is **ignored** when comparing precedence.
- Precedence order: `1.0.0-alpha` < `1.0.0-alpha.1` < `1.0.0-beta` < `1.0.0-rc.1` < `1.0.0`.

**npm range operators** (what your `package.json` dependencies actually mean):

| Range | Matches | Notes |
|---|---|---|
| `^1.4.2` | `>=1.4.2 <2.0.0` | Caret — default on `npm install`; allows minor + patch |
| `~1.4.2` | `>=1.4.2 <1.5.0` | Tilde — allows patch only |
| `1.4.2` | exactly `1.4.2` | Pinned |
| `*` / `latest` | anything | Avoid in production |

> Caret (`^`) is why `0.x` is special in npm too: `^0.4.2` resolves to `>=0.4.2 <0.5.0`
> (patch only), because pre-`1.0.0` every minor is treated as potentially breaking.

---

## How Conventional Commits drive SemVer

This mapping is the whole point of Conventional Commits — it lets tools compute the next version and generate the changelog **automatically**.

| Commit | Triggers | Example bump |
|---|---|---|
| `fix: ...` | **PATCH** | `1.4.2` → `1.4.3` |
| `feat: ...` | **MINOR** | `1.4.2` → `1.5.0` |
| any type + `!` or `BREAKING CHANGE:` | **MAJOR** | `1.4.2` → `2.0.0` |
| `docs`, `chore`, `test`, `ci`, ... | usually **no release** | — |

So a release since the last tag containing three `fix`, one `feat`, and one `BREAKING CHANGE` produces a **major** bump — the highest-ranked change wins.

---

## Node.js tooling

The JS ecosystem has the richest automation around this convention. A typical setup:

### 1. Lint commit messages — `commitlint` + `husky`

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional husky
npx husky init
echo 'npx --no -- commitlint --edit "$1"' > .husky/commit-msg
```

`commitlint.config.js`:

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  // Optional: teach it your custom types/scopes
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'refactor', 'perf', 'test',
       'build', 'ci', 'chore', 'style', 'revert', 'a11y'],
    ],
  },
};
```

Now a malformed message is rejected before it's even committed:

```
$ git commit -m "fixed the search bug"
⧗   input: fixed the search bug
✖   type must be one of [feat, fix, ...] [type-enum]
✖   found 1 problem, 0 warnings
```

### 2. Guided prompts (optional) — `commitizen`

```bash
npm install --save-dev commitizen cz-conventional-changelog
npx commitizen init cz-conventional-changelog --save-dev --save-exact
# then commit with:
npx cz
```

### 3. Automated releases — pick one

**`semantic-release`** — fully automated: analyzes commits, bumps version, writes the changelog, tags, and publishes to npm + GitHub, all in CI. Zero manual version editing.

```bash
npm install --save-dev semantic-release
```
```yaml
# .github/workflows/release.yml (excerpt)
- run: npx semantic-release
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**`changesets`** — popular for **monorepos**; contributors add an intent file per change and versions are batched at release time. Less "magic", more control.

**`commit-and-tag-version`** — the maintained fork of the now-retired `standard-version`. Local, no CI: one command does the whole cut — derive the next version from the commits since the last `v*` tag, then write `CHANGELOG.md`, the release commit, and the tag. The best middle ground when you release by hand but still want the version and changelog derived from history.

```bash
npm install --save-dev commit-and-tag-version
```
```jsonc
// package.json — wire it to a script
"scripts": { "release": "commit-and-tag-version" }
```
```bash
npm run release                             # feat -> minor, fix -> patch, ! -> major
npx commit-and-tag-version --dry-run        # preview the bump + notes, write nothing
npx commit-and-tag-version --first-release  # first cut: tag current version, don't bump
git push --follow-tags                      # publish the release commit and its tag
```

**Shape the changelog** with `.versionrc.json` — map each type to a heading, or hide it so housekeeping never clutters the notes:

```json
{
  "header": "# Changelog\n\nFormat: Conventional Commits · Versioning: SemVer\n",
  "types": [
    { "type": "feat",     "section": "Features" },
    { "type": "fix",      "section": "Bug Fixes" },
    { "type": "perf",     "section": "Performance" },
    { "type": "refactor", "section": "Refactoring" },
    { "type": "revert",   "section": "Reverts" },
    { "type": "docs",  "hidden": true },
    { "type": "chore", "hidden": true },
    { "type": "ci",    "hidden": true }
  ]
}
```

The generated `CHANGELOG.md` groups commits under those headings, links each to its hash, and takes the version from the highest-ranked change in the range:

```markdown
## [1.5.0](https://github.com/you/repo/compare/v1.4.2...v1.5.0) (2026-07-04)

### Features
* **search:** add filter by category (a1b2c3d)

### Bug Fixes
* **search:** return 400 on invalid filter values (d4e5f6a)
```

> This is the auto-changelog payoff of Conventional Commits: you write the commits, the tool writes the release notes. Hidden types (`docs`, `chore`, `ci`, …) are still committed — they just don't appear in the notes or move the version.

### 4. Plain npm (no extra tooling)

`npm version` bumps `package.json`, commits, and tags in one step:

```bash
npm version patch   # 1.4.2 -> 1.4.3
npm version minor   # 1.4.2 -> 1.5.0
npm version major   # 1.4.2 -> 2.0.0
npm version prerelease --preid=beta   # 1.5.0 -> 1.5.1-beta.0
```

---

## The first commit

The spec says **nothing** about the first commit — it's not special. In practice:

- **`Initial commit`** is the de-facto default — it's literally what **GitHub auto-generates** when you create a repo "with a README". (Amusingly, it violates the lowercase convention itself.)
- In **strict Conventional Commits repos**, it's usually shaped to fit:

```
chore: init
chore(repo): initialize project
feat: initial release       # only if the first commit ships working functionality
```

`chore` is the most logical type — scaffolding a repo doesn't concern users. Some teams configure `commitlint` to **ignore the very first commit** so it can be named anything.

---

## Myth-busting: what the spec actually requires

The gap between "the specification" and "the habit" trips people up constantly.

- **Only `feat` and `fix` are defined by the spec.** All the other types (`build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`) come from the **Angular convention** and the `commitlint` preset — they're conventional, not mandated, and by themselves don't drive SemVer.
- **lowercase is NOT a spec requirement.** The spec treats commit parts as **case-insensitive**, *except* `BREAKING CHANGE`, which must be uppercase. The all-lowercase habit (`feat:`, no capital, no trailing period) is **Angular style**, adopted wholesale — but not part of the required core. React proves the point: `[Fizz] Fix non-script modulepreload tracking` uses a capital `Fix`.
- **Where it comes from:** the `feat`/`fix`/`docs`/... vocabulary originated in **AngularJS**'s contributing guide. The **Conventional Commits** spec was created by **Benjamin E. Coe** (of the Yargs/npm ecosystem) in **2017**, to automate version bumping and changelog management.

---

## Reality check: what big projects really do

Pulled from real repositories — the landscape splits into two camps.

**Use Conventional Commits (strictly):**

*Angular* (the originator):
```
feat(migrations): add migration from injectable to service
fix(docs-infra): remove white flash on example viewer tab labels
build: update pnpm to v11.8.0
```

*Vue* (with automated releases):
```
fix(compiler-core): avoid crash on CDATA at the document root (#14916)
chore(deps): update all non-major dependencies (#14938)
release: v3.5.38
```

**Do NOT — each has its own system:**

*React* (Meta) — bracket-tag area instead of `type:`:
```
[Fizz] Fix non-script modulepreload tracking (#36564)
[compiler] Count loop reassignments as the enclosing scope reassigning the variable
```

*Kubernetes* — mostly free-form imperative:
```
Return a runtime-specific error when NUMA distances are unavailable
Remove unused scheduling queue methods
```

*Linux* — `subsystem: description` + mandatory `Signed-off-by:` (DCO):
```
erofs: handle 48-bit blocks_hi for compressed inodes

Signed-off-by: Jane Doe <jane@example.com>
```

*VS Code* (Microsoft) — free-form, sometimes with a colon-area:
```
sessions: show newly started local sessions in the list immediately (#322471)
Respect workspace trust for the remote agent host (#322459)
```

> **Takeaway:** Conventional Commits is the most popular *named* convention — dominant in the
> npm/JS ecosystem and among libraries that want automatic changelogs and SemVer releases. But
> it is **not** an industry-wide mandate. Half of the world's largest projects use their own
> rules. What's truly universal isn't the prefixes — it's the discipline: a clear imperative
> header, a narrow scope, and the 50/72 rule.

---

## Other conventions

Roughly from freest to strictest:

1. **Plain imperative ("50/72" / GitHub style)** — just a clear command header. Zero overhead, nothing to learn; but no changelog/version automation.
   ```
   Add search endpoint
   Fix off-by-one error in pagination
   ```
2. **Conventional Commits** — `type(scope): description`. Automatic changelog + SemVer; needs a linter to hold together. Best for libraries and published packages.
3. **Angular convention** — CC's stricter parent (fixed scope list, mandated body). Rarely adopted whole; most people take the softer CC.
4. **Linux kernel style** — `subsystem: description` + mandatory `Signed-off-by:` (DCO — legal proof of authorship). Required to contribute to the kernel.
5. **Bracket-tag style** — area in brackets instead of `type:` (React's `[Fizz] Fix ...`). A house convention, not standardized.
6. **gitmoji** — emoji as (or with) the type: `✨ feat: add dark mode`. Visual, niche, renders poorly in some terminals.
7. **Ticket-prefixed (enterprise)** — `[JIRA-1234] Add login throttling`. Dominant in closed corporate repos; rare in open source (external contributors have no tracker access).

---

## What should *you* use?

> **The format should match whether you need automation.**

- **Public library / published package?** → **Conventional Commits.** You get a free changelog and automatic SemVer, and it's the one named convention contributors likely already know. Add `commitlint` + `husky` so the format doesn't drift.
- **App / personal project / internal tool / script?** → **plain imperative + 50/72 discipline.** No changelog or SemVer needed, so don't over-engineer. It's honest, readable, and free.
- **Contributing to someone else's project?** → the choice is already made for you. **Open `CONTRIBUTING.md`** and match it. That overrides every personal preference — write `fix(compiler-core):` for Vue, `[Fizz] Fix ...` for React.

---

## One-page cheat card

```
FORMAT
  <type>(<scope>)<!>: <summary>          # header (required)
                                         # blank line
  <body>                                 # why (optional, wrap at 72)
                                         # blank line
  <footer>                               # Closes #, BREAKING CHANGE:, Co-authored-by:

TYPES        feat fix docs refactor perf test build ci chore style revert
             (spec mandates only feat + fix)

SEMVER       fix    -> PATCH   1.4.2 -> 1.4.3
             feat   -> MINOR   1.4.2 -> 1.5.0
             ! / BREAKING CHANGE -> MAJOR  1.4.2 -> 2.0.0

SUMMARY      imperative  •  ≤50 chars  •  lowercase  •  no trailing period
SCOPE        kebab-case  •  consistent  •  matches folder structure
FOOTER       Closes #482  •  BREAKING CHANGE: ...  •  Co-authored-by: Name <email>

GOLDEN RULE  Header = WHAT.  Body = WHY.  When in doubt, read CONTRIBUTING.md.
```

---

## Sources

- **Conventional Commits 1.0.0** — <https://www.conventionalcommits.org/en/v1.0.0/>
- **Semantic Versioning 2.0.0** — <https://semver.org/>
- **Angular commit message guidelines** (the origin of the type vocabulary) — <https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md>
- **commitlint** — <https://commitlint.js.org/>
- **semantic-release** — <https://semantic-release.gitbook.io/>
- **commit-and-tag-version** — <https://github.com/absolute-version/commit-and-tag-version>
- **Changesets** — <https://github.com/changesets/changesets>

---

## License

Documentation is released under
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
Code snippets are released under the [MIT License](https://opensource.org/licenses/MIT).
