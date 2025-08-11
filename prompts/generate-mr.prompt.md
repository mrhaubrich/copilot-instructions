---
mode: 'agent'
tools: ['githubRepo', 'runCommands', 'search', 'searchResults', 'terminalLastCommand', 'usages', 'codebase']
description: 'Generate a GitHub/GitLab PR title and description from a diff'
---

Generate a **pull request title and description** based on the code changes between two branches.

### 1. Identify Git Provider

Run:  
```bash
git config --get remote.origin.url
```

- If the URL contains `"github.com"`, assume **GitHub**.  
- If it contains `"gitlab.com"`, assume **GitLab**.  

This determines whether to follow GitHub or GitLab markdown and PR conventions.

### 2. Detect Language

Check if the project is written in English or Portuguese:

- Inspect the contents of `README.md` or look for keywords to determine if the default language is **Brazilian Portuguese** (`pt-BR`).  
- If unclear, default to **pt-BR**.  

The PR title and description must be written in the same language as the project.

### 3. Generate the Diff

Use the following command to compute the diff from the merge base of the target and source branches and write it to a file to avoid interactive paging:

```bash
git diff $(git merge-base ${input:targetBranch} ${input:sourceBranch}) ${input:sourceBranch} > diff.patch
```

Read the contents of `diff.patch` to summarize the changes.
You can read the contents without using `cat` or `less` by using the `readFile` function.

### 4. Format the Output

Generate a **PR title** and **description**:

- **Title**  
  - Max 50 characters  
  - Imperative mood  
  - Conventional commit prefix (`feat:`, `fix:`, `chore:`, etc.)  
  - No trailing period  
  - Should use gitmoji for the prefix if possible.
  - It could have the scope of the change, if applicable.
  - Example: `feat(auth): :sparkles: Add token-based login flow`  

- **Description**  
  Should include the following sections:
  1. **What** - Briefly describe what changed
  2. **Why** - Explain the rationale and context
  3. **Context & Problem** - Describe the context and the problem being solved
  4. **Solution Overview** - Bullet list of major changes
  5. **Verification & Testing** - Testing steps and verification methods
  6. **Impact & Rollout** - Migration notes, feature flags, rollback plan
  7. **Related Issues** - Link to related issues/tickets
  8. **Reviewers** - Tag reviewers if applicable
  9. **Labels** - Suggested labels for the PR

Write the description in clear Markdown.

### 5. Output Format

Return the result as a fenced code block labelled `markdown`. For example:

    ```markdown
            ## Title
        <!-- e.g. Feature: Add JWT authentication endpoint -->

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
        1. ...
        2. ...

        ## Impact & Rollout
        - **Migrations:** 
        - **Feature Flags:** 
        - **Rollback Plan:** 

        ## Related Issues
        Closes #...

        ## Reviewers
        /assign_reviewer @username1 @username2

        ## Labels
        backend, high-priority

    ```

Ensure the entire PR title and description are inside this single code block.

### 6. Remove the Diff File
After generating the PR title and description, delete the `diff.patch` file to clean up.

```bash
rm diff.patch
```
