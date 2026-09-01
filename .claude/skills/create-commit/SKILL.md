---
name: create-commit
description: Create a conventional commit message for staged changes by analyzing the diff and the project structure to pick the proper type and scope. Use when the user asks to commit, write a commit message, or before running git commit.
---

# Create commit

## Gather context

1. Run `git status --short` and `git diff --cached --stat` to see what is staged. If nothing is staged, ask the user what to include; never stage files yourself.
2. Run `git diff --cached` to read the actual changes.
3. Identify the project structure to ground type and scope. Detect the project kind from its manifest: `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `Gemfile`, `composer.json`, `*.csproj`, `pom.xml`, `build.gradle`. Note workspaces/monorepo layout (npm workspaces, Cargo members, pnpm, turborepo) and treat each package or crate as a scope candidate. Note framework or domain folders (src, lib, api, web, docs) as secondary scope hints.
4. Run `git log --oneline -10` to match the message style already used in the repo.

## Determine type

Identify the product first: the source a user actually runs or consumes — the main script, binary, library, or app code. Derive it from the manifest entry points and the layout found in Gather context. In a single-script project, that script is the product.

Then pick the first rule that matches the intent of the staged diff:

- `feat` — adds behavior or a capability to the product, however small the diff: a new flag, step, output, or surface
- `fix` — corrects product behavior; a bug fix
- `perf` — product performance improvement without behavior change
- `refactor` — product code moved, renamed, or restructured, no behavior change
- `docs` — documentation only
- `test` — adding or correcting tests only
- `build` — dependency, manifest, or build-system changes
- `ci` — pipeline or workflow configuration
- `chore` — work that never changes what the product does: repo metadata, tooling, configs, generated files
- `revert` — reverting a previous commit

For product code, classify by the effect on the person running the product, not by diff size. A two-line addition to the product that gives users something new is `feat`, never `chore`. `chore` is not a default for small or hard-to-classify diffs.

Mixed changes: if the staged diff contains clearly independent areas, propose splitting them into separate commits instead of mixing them. Only commit mixed areas together when they genuinely serve one intent, and pick the type from the product effect described above — not from which area has the most lines.

## Determine scope

- Infer the scope from the changed file paths: prefer the workspace, package, or crate name that owns the files; fall back to the top-level directory (for example `api`, `web`, `cli`).
- Omit the scope entirely when there is only one possible owner — a single-package project with one top-level area gains nothing from a scope. A scope is only useful when it distinguishes among several candidates, as in a monorepo or a project with clearly separate areas.
- Omit the scope entirely when the change spans many areas or no clear owner exists.
- Never write empty parentheses — use `type: subject`, not `type(): subject`.

## Compose the message

Format: `type(scope): subject` or `type: subject`.

- All lowercase in the header: type, scope, and subject must not contain capitals. The hooks reject them.
- Imperative mood: `add`, `fix`, `support` — not `added`, `fixes`, `adding`.
- No trailing period in the subject.
- Keep the subject short: aim for 50 characters, hard limit 100 for the whole header.
- Never write a body. The message is the header alone: one line, no blank line, nothing after it. If extra explanation seems needed, put it in the code or leave it out — the commit stays bodyless.
- Use plain English words in the message; misspellings, made-up terms, and unusual identifiers are rejected.

## Commit

1. Stage nothing extra. Commit with the composed message.
2. If the commit-msg hook rejects the message, read its error (unknown word, case, or conventional-commit problem), fix the message, and retry.
3. Never use `--no-verify` or otherwise bypass the hooks.
