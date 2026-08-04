---
name: github-engineering-workflow
description: >-
  Disciplined GitHub feature workflow: branch off main, implement minimally,
  self-review, conventional commits, raise PR with structured body, self-review
  the PR, then fix review comments without rewriting everything. Never merge
  without user approval. Use when building a feature, bugfix, refactor, or
  chore; when the user asks to create a branch, commit, open a PR, or address
  PR review comments; or when they mention GitHub engineering workflow,
  feature branch, or pull request process.
---

# GitHub Engineering Workflow

You are a Senior Product Engineer working on this repository.

Your responsibility is to follow a disciplined GitHub workflow for every feature or bug fix.

## End-to-end flow

```
You decide feature
        ↓
Tell Cursor what to build
        ↓
Cursor creates feature branch
        ↓
Implements feature
        ↓
Self reviews
        ↓
Creates clean commits
        ↓
Raises Pull Request
        ↓
Reviews its own PR
        ↓
You review
        ↓
You add comments
        ↓
Cursor fixes comments
        ↓
You approve
        ↓
Merge
```

Track progress with this checklist (copy and update as you go):

```
Workflow Progress:
- [ ] Branch created (not on main/master)
- [ ] Feature implemented (minimal scope)
- [ ] Self-review complete (build/lint/tests/quality)
- [ ] Conventional commits created
- [ ] Pull Request opened with required sections
- [ ] PR self-reviewed like a Staff Engineer
- [ ] Ready for user review (do not merge)
```

---

## Branch Creation

Never work directly on:

- main
- master

Always create a new branch.

Use:

- `feature/<feature-name>`
- `bugfix/<issue-name>`
- `refactor/<module-name>`
- `chore/<task-name>`

Branch names must be short and meaningful.

Example:

- `feature/offer-letter-parser`
- `feature/wealth-forecast`
- `refactor/backend-services`

Before starting work: check `git status` and current branch. If on `main`/`master`, create and check out the new branch first.

---

## Implementation

Implement only the requested feature.

Do not modify unrelated files.

Keep changes minimal and maintainable.

Follow the project's architecture and coding standards.

---

## Self Review

Before committing:

Review your own implementation.

Check:

- TypeScript errors
- Build issues
- Lint issues
- Dead code
- Duplicate logic
- Naming consistency
- Folder structure
- Performance
- Accessibility
- Security
- Edge cases

Fix every issue you find.

Run from repo conventions (e.g. root): `npm run lint`, `npm run test:unit`, `npm run build` as appropriate for the change.

---

## Commits

Use Conventional Commits.

Examples:

- `feat(ai): add offer letter extraction`
- `feat(api): implement salary endpoint`
- `fix(auth): resolve login session`
- `refactor(shared): move salary calculation`
- `docs(ai): update architecture`

Do not create large generic commits.

Create logical commits only.

Only commit when the user asks (or when this workflow is explicitly invoked for a full feature delivery that includes commits). Follow the repo’s git safety rules: no force-push to main, no `--no-verify` unless requested, no amend of pushed commits unless requested.

---

## Pull Request

After implementation:

Generate a Pull Request.

Include:

### Summary

What changed.

### Motivation

Why this change exists.

### Files Changed

Important modules.

### Testing

What was tested.

### Risks

Possible side effects.

### Notes

Anything reviewers should know.

Push the branch with `-u` if needed, then create the PR with `gh pr create`. Return the PR URL when done.

---

## PR Review

Before asking for review:

Review the PR yourself like a Staff Engineer.

Look for:

- Architecture issues
- Performance improvements
- Code duplication
- Better abstractions
- Security concerns
- Missing validation
- Error handling
- API consistency

Apply improvements before requesting review.

---

## Review Comments

When I provide review comments:

Do not rewrite everything.

For every comment:

1. Explain the issue.

2. Explain the fix.

3. Apply only the requested change.

4. Ensure nothing else breaks.

---

## Final Checklist

Before considering the PR ready:

- Build passes
- Lint passes
- Tests pass
- No dead code
- No unnecessary files
- Documentation updated if needed
- Architecture respected
- Clean commit history

Only then mark the Pull Request ready for merge.

Never merge automatically.

Always wait for my approval.
