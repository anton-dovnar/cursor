---
name: commit-conventions
description: Generate conventional commit messages following project standards. Use before running git commit to analyze staged changes and create properly formatted commit messages with type, scope, summary, and body. Triggers on git commit workflows or when commit messages need to be created.
---

# Commit Conventions

## Workflow

Before committing:

1. Run `git diff --staged` to review all staged changes
2. Analyze changes to determine type, scope, and impact
3. Generate commit message using format below
4. Follow Conventional Commits standard

---

## Required Commit Message Format

All commit messages MUST follow **Conventional Commits**.

### Allowed types:

- feat: new features
- fix: bug fixes
- refactor: code restructuring without user-visible change
- docs: documentation updates
- style: formatting, whitespace, no code changes
- test: add or update tests
- chore: maintenance tasks
- perf: performance improvements
- build: build system changes
- ci: CI/CD pipeline updates

### Choosing the Right Type

**Decision tree:**

- New user-facing functionality? → `feat`
- Fixing broken behavior? → `fix`
- Code restructuring, no behavior change? → `refactor`
- Test changes only? → `test`
- Documentation only? → `docs`
- Dependencies, tooling, build? → `chore`
- Performance improvement? → `perf`
- CI/CD changes? → `ci`

**Common scopes in this project:**

- Better Auth changes → `feat(auth)` or `fix(auth)`
- Supabase schema → `feat(db)` or `refactor(db)`
- React components → `feat(ui)`, `fix(ui)`, `refactor(ui)`
- API endpoints → `feat(api)`, `fix(api)`
- Web package → `feat(web)`, `fix(web)`
- Landing page → `feat(landing)`, `fix(landing)`

### Structure (Conventional Commit Standard):

A commit message MUST follow this structure:

    <type>(optional scope): <short summary in present tense>

    <optional body>
      - Explain what changed
      - Explain why it changed
      - List affected components/files
      - Provide important context

### Summary Line Requirements:

- Maximum ~50 characters recommended
- Use present-tense imperative mood
- Be specific and descriptive

### Body Requirements:

- Explain WHAT changed
- Explain WHY it changed
- Include affected files/components
- Provide relevant context

---

## Analyzing Staged Changes

When reviewing `git diff --staged`:

1. **Identify primary change:** What's the main purpose?
2. **Determine scope:** Which package/module? (api, web, shared, landing)
3. **Check for multiple concerns:** If changing unrelated areas, consider splitting
4. **List affected files:** Include 2-3 most important files in body
5. **Capture the "why":** Why was this change necessary?

---

## Best Practices

- Use clear and precise commit messages
- Prefer scopes for clarity (e.g. feat(api), fix(ui))
- Start summary with action verb
- Include the “why,” not just the “what”
- Never commit vague messages

---

## Forbidden Elements

- NEVER use generic messages such as “update files” or “fix issues”

---

## Examples

### Feature

```
feat(auth): add session persistence across browser restarts

- Implement Better Auth session cookie refresh
- Add session validation middleware
- Update auth-client.ts with auto-refresh logic
```

### Fix

```
fix(chat): prevent message duplication on network retry

- Add request deduplication in Chat.tsx
- Implement idempotency key for POST /api/chat
- Affected: packages/web/src/components/Chat.tsx, packages/api/src/routes/chat.routes.ts
```

### Refactor

```
refactor(api): extract database queries into service layer

- Move queries from controllers to database.service.ts
- Add type-safe query wrapper methods
- Improve error handling consistency
```

### Multi-package

```
feat(chat): add voice input support

- Integrate OpenAI Whisper API in packages/api
- Implement useVoiceInput hook in packages/web
- Add VoiceButton component with recording UI
```
