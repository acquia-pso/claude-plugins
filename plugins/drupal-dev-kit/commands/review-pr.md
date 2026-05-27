---
name: review-pr
description: Review the current PR/MR for coding standards and security issues
---

# Review PR Command

This command reviews the diff of the current open PR/MR and produces a structured review ready to post as a comment.

## Execution Steps

1. **Fetch changed file list** — start cheap, no diff content yet
   - Local (preferred): `git --no-pager diff --name-status origin/main...HEAD`
   - Branch not local — GitHub: `gh pr view 123 --json files --jq '.files[].path'`
   - Branch not local — GitLab: `glab api projects/:fullpath/merge_requests/123/changes --jq '.changes[].new_path'`
   - Use `--name-status` (not `--name-only`) to get change type (A/M/D/R) — needed to skip deleted files in step 2
   - See `skills/git/references/diff-practices.md` for full guidance

2. **Identify changed PHP files**
   - Filter the file list to `.php` files only
   - Skip any files marked `D` (deleted) — `git show` and `git diff` will fail on deleted files
   - If the branch **is** local: read each file via `git --no-pager diff origin/main...HEAD -- path/to/file.php` (changed hunks) or `git show HEAD:path/to/file.php` (full file for new/added files)
   - If the branch **is not** local: prefer checking it out via the platform CLI first — `gh pr checkout 123` (GitHub) / `glab mr checkout 123` (GitLab) — and then use local git commands. If you use manual git commands instead, `git fetch origin branch-name && git checkout branch-name` only works when the source branch exists on `origin`; for fork PRs/MRs, add/fetch the fork remote first. Otherwise, retrieve per-file content from the platform API: `GH_PAGER=cat gh pr diff 123 -- path/to/file.php` (GitHub) / `PAGER=cat glab mr diff 123` (GitLab)
   - Only save a full diff to a temp file if you need to scan many files at once (see Diff Best Practices)

3. **Invoke drupal-code-reviewer**
   - Pass the list of changed PHP files
   - Reviewer will check for coding standards violations and security issues

4. **Produce structured review**
   - Summarize review findings
   - Format as a PR/MR comment
   - Include:
     - Summary (files reviewed, issues found)
     - Security issues (if any)
     - Coding standards violations (if any)
     - Approved files (if any)

5. **Output the review**
   - Return the formatted review as markdown
   - Do NOT automatically post the comment (let developer review it first)
   - Provide instructions for posting: `gh pr comment <PR#> --body "$(cat review.md)"` or `glab mr note <MR#> --message "$(cat review.md)"`

## Output Format

```markdown
## Code Review

**Reviewed by**: Claude Code
**Date**: 2026-04-16
**PR/MR**: #123

### Summary

**Files Reviewed**: 5
**Issues Found**: 2 (1 security, 1 standards)

---

### Security Issues

#### web/modules/custom/acme/src/Controller/UserController.php:45
**Issue**: SQL injection vulnerability
**Problem**: Raw SQL query without placeholders
**Fix**: Use query builder or parameterized query

---

### Coding Standards Violations

#### web/modules/custom/acme/src/Form/SettingsForm.php:12
**Issue**: Direct use of `\Drupal::config()`
**Problem**: Violates dependency injection pattern
**Fix**: Inject `config.factory` service via constructor

---

### Approved

- web/modules/custom/acme/acme.module (hooks follow conventions)
- web/modules/custom/acme/src/Service/UserService.php (clean DI, proper typing)
- web/modules/custom/acme/acme.services.yml (correct service definition)
```

## Example Usage

User says: "/review-pr"

You respond:
1. Fetch PR diff via git-manager
2. Identify changed PHP files
3. Spawn drupal-code-reviewer
4. Format review output
5. Provide posting instructions

## TODO

- Project-specific review criteria may need to be added
- Custom linting rules may vary
