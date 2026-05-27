# Git Diff Best Practices

Diffs can be arbitrarily large. The goal is to avoid two failure modes: **pager hang** (shell blocks waiting for input) and **context flood** (entire diff dumped into context at once).

## Prefer local git over `gh`/`glab` diff

Local `git` is faster, works offline, and supports filtering flags (`--name-only`, `--stat`, `-- path`). Use `gh pr diff` / `glab mr diff` only when the branch isn't available locally (e.g. an unmerged fork PR).

```bash
# Preferred — local, filterable
# Use origin/main (not main) so the comparison is against the actual remote base,
# regardless of whether local main is up to date
git --no-pager diff origin/main...HEAD
git --no-pager diff --name-only origin/main...HEAD
git --no-pager diff --stat origin/main...HEAD
git --no-pager diff origin/main...HEAD -- path/to/file.php

# Fallback — branch not local
GH_PAGER=cat gh pr diff 123      # gh uses GH_PAGER, not GIT_PAGER
PAGER=cat glab mr diff 123       # glab uses PAGER
```

## Get file names first, content second

Start cheap. File names and stats are tiny — read them directly. Only fetch diff content when you know which files you need.

```bash
# File names only (local)
git --no-pager diff --name-only origin/main...HEAD
git --no-pager diff --name-status origin/main...HEAD   # includes A/M/D/R type

# Stats (local)
git --no-pager diff --stat origin/main...HEAD

# File names via API — no diff content at all (useful when branch not local)
gh pr view 123 --json files --jq '.files[].path'                                          # GitHub
glab api projects/:fullpath/merge_requests/123/changes --jq '.changes[].new_path'        # GitLab
```

## Read content selectively

**`git diff -- file` vs `git show`**: For a *modified* file, `git diff -- file` only outputs the changed hunks — typically 40–60% smaller than `git show`, which outputs the entire file. Prefer `git diff -- file` when reviewing changes; use `git show` only when you need the full file for context.

```bash
# Reviewing what changed — use diff (smaller)
git --no-pager diff origin/main...HEAD -- path/to/file.php

# Need the full file (e.g. new file, or need surrounding context)
git show HEAD:path/to/file.php
```

If `--stat` shows > ~500 lines changed, save to a temp file rather than reading directly.

```bash
DIFF_FILE=$(mktemp /tmp/diff.XXXXXX)
git --no-pager diff origin/main...HEAD > "$DIFF_FILE"
# or when branch not local:
# GH_PAGER=cat gh pr diff 123 > "$DIFF_FILE"
# PAGER=cat glab mr diff 123 > "$DIFF_FILE"

wc -l "$DIFF_FILE"                          # how big is it?
grep '^diff --git' "$DIFF_FILE"             # which files?

# Extract one file's chunk
awk '
  /^diff --git a\/path\/to\/file\.php b\/path\/to\/file\.php$/ { printing=1 }
  printing && /^diff --git / && $0 !~ /^diff --git a\/path\/to\/file\.php b\/path\/to\/file\.php$/ { exit }
  printing { print }
' "$DIFF_FILE"

rm "$DIFF_FILE"
```

## Disable the pager

`git`, `gh`, and `glab` may open `less` or `$PAGER`, which hangs the shell. Always suppress it:

```bash
git --no-pager diff ...           # git flag (simplest)
GH_PAGER=cat gh pr diff 123      # gh uses GH_PAGER
PAGER=cat glab mr diff 123       # glab uses PAGER
```

## Verify a branch has changes before diffing

A branch may already be merged or empty. Always check first to avoid wasted work:

```bash
git --no-pager log --oneline origin/main...origin/branch-name
# No output = nothing to diff (already merged or no unique commits)
```

## Decision tree

```
Need diff content?
│
├─ Does the branch have unmerged commits?
│   └─ git --no-pager log --oneline origin/main...origin/branch-name
│      (no output = already merged, stop here)
│
├─ Just file names / stats?
│   └─ git --no-pager diff --name-only origin/main...HEAD        (local)
│      git --no-pager diff --stat origin/main...HEAD             (local)
│      gh pr view 123 --json files --jq '.files[].path'          (GitHub, branch not local)
│      glab api ...merge_requests/123/changes --jq '.changes[].new_path'   (GitLab)
│
├─ Specific file(s), reviewing changes?
│   └─ git --no-pager diff origin/main...HEAD -- path/to/file    (prefer over git show — hunks only)
│
├─ Need full file content (new file or broad context)?
│   └─ git show HEAD:path/to/file
│
└─ Full diff or stat shows > ~500 lines changed?
    └─ Save to temp file, then read selectively
       DIFF_FILE=$(mktemp /tmp/diff.XXXXXX)
       git --no-pager diff origin/main...HEAD > "$DIFF_FILE"
```
