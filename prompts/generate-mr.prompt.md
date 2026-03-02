---
agent: agent
name: generate-mr
description: Generate a GitHub or GitLab pull/merge request title and description from a diff between two branches.
tools: [vscode/runCommand, vscode/askQuestions, execute, read, agent, search/changes]
---

Generate a **pull request title and description** based on the code changes between two branches.

### 1. Identify Git Provider

Use `vscode/runCommand` to run:

```bash
git config --get remote.origin.url
```

- If the URL contains `"github.com"`, assume **GitHub**.  
- If it contains `"gitlab.com"`, assume **GitLab**.  

This determines whether to follow GitHub or GitLab markdown and PR conventions.

If the command fails or no remote is configured, default to GitHub and continue.

### 2. Detect Language

Use `search/changes` and `read` to inspect `README.md` and other top-level docs to detect if the project is written in English or Portuguese.

- If the default language appears to be **Brazilian Portuguese** (`pt-BR`), write the MR in pt-BR.  
- If unclear, default to **pt-BR**.  

The PR title and description must be written in the same language as the project.

### 3. Generate and Read the Diff

Ask the user for the branches if not provided yet using `vscode/askQuestions`:

- `targetBranch` (base, for example, `main`)
- `sourceBranch` (feature branch)

Use `vscode/runCommand` or `execute` to compute the diff from the merge base and write it to a temporary file to avoid interactive paging:

```bash
git diff "$(git merge-base ${input:targetBranch} ${input:sourceBranch})" "${input:sourceBranch}" > diff.patch
```

Then use `read` to read the contents of `diff.patch` and summarize the changes.

When summarizing the diff:

- Focus on **logical changes**: features, fixes, refactors, config changes, schema migrations.
- De-emphasize low-signal changes: formatting-only adjustments, re-generated code, lockfiles, etc.

### 4. Format the Output

Generate a **PR title** and **description**.

#### Title

The title must:

- Use a **Conventional Commit** prefix: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, or `perf:`.  
- Optionally include a **scope** in parentheses, for example `feat(auth):`, `fix(api):`, `chore(ci):`.  
- Be written in **imperative mood** (for example, “add”, “fix”, “update”).  
- Have **no trailing period**.  
- Be at most **50 characters** when possible, but favor clarity over strict length.  
- Prefer a matching **gitmoji** prefix when it improves readability:
  - `:sparkles:` for new features (`feat:`)
  - `:bug:` for bug fixes (`fix:`)
  - `:recycle:` for refactors (`refactor:`)
  - `:memo:` for documentation (`docs:`)
  - `:white_check_mark:` for tests (`test:`)

Example (adapt wording and language to the repo):

```text
feat(auth): :sparkles: Add token-based login flow
```

#### Description

Write the description in clear Markdown, in the same language as the repository.

It should include the following sections:

1. **What** – Briefly describe what changed.
2. **Why** – Explain the rationale and context.
3. **Context & Problem** – Describe the context and the problem being solved.
4. **Solution Overview** – Bullet list of major changes.
5. **Verification & Testing** – Testing steps and verification methods.
6. **Impact & Rollout** – Migration notes, feature flags, rollback plan.
7. **Related Issues** – Links to related issues/tickets.
8. **Reviewers** – Suggested reviewers to tag.
9. **Labels** – Suggested labels for the PR/MR.

Guidelines:

- Prefer concrete, high-signal bullets derived from the diff.  
- Do not hallucinate business context; if it is not obvious from the changes, add a short, neutral placeholder for the author to fill in (in the correct language).  
- Call out database migrations, config changes, and breaking changes explicitly.  

### 5. Output Format

Return the result as a fenced code block labelled `markdown`. Do not put any text outside this block.

Follow this structure (translate headings and inline comments to the detected language):

```markdown
```markdown
## Title
feat(auth): :sparkles: Add token-based login flow

## What
- Briefly describe what changed.

## Why
- Explain the rationale and context.

## Context & Problem
- **Context:**
- **Problem:**

## Solution Overview
- Bullet list of major changes.

## Verification & Testing
- [ ] Unit tests added/updated
- [ ] Integration tests
- [ ] Manual steps:
  - [ ] Step 1
  - [ ] Step 2

## Impact & Rollout
- **Migrations:**
- **Feature Flags:**
- **Rollback Plan:**

## Related Issues
- Closes #...

## Reviewers
- @username1
- @username2

## Labels
- backend
- high-priority
```
```

Ensure the entire PR title and description are inside this single `markdown` code block.

### 6. Clean Up

After generating the PR title and description, use `vscode/runCommand` or `execute` to remove the temporary diff file:

```bash
rm -f diff.patch
```

If cleanup fails, do not block returning the generated MR content; the text output has priority.
