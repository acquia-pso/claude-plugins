---
name: git
description: Git, GitHub (gh), GitLab (glab), CI operations. Branching, commits, PRs/MRs, diffs, CI. Triggers on git, PR, MR, branch, diff.
compatibility: Designed for Claude Code. Requires git, gh or glab CLI.
---

# Git CLI Skill

Platform is determined by the `GIT_PLATFORM` environment variable (`github` or `gitlab`).

## Reference files

| Activity | Reference | Use when |
|---|---|---|
| Branching, naming, renaming | [references/branch-operations.md](references/branch-operations.md) | Creating branches, following naming conventions, or renaming a branch safely and verifying/recovering PR state |
| Diffing safely | [references/diff-practices.md](references/diff-practices.md) | Reviewing changes between branches — avoids pager hangs and context floods on large diffs |

## Platform detection

```bash
if [[ "$GIT_PLATFORM" == "github" ]]; then
  # Use gh
elif [[ "$GIT_PLATFORM" == "gitlab" ]]; then
  # Use glab
fi
```

## GitHub (`gh`)

**Authenticate**: `gh auth login`

**Create PR** — always use `--body-file` with a temp file (inline `--body` breaks on backticks, variables, newlines):
```bash
BODY_FILE=$(mktemp /tmp/pr-body.XXXXXX)
cat > "$BODY_FILE" << 'EOF'
## Ticket
PROJ-123

## Summary
Brief description.

## Testing Steps
1. Step one
EOF

if gh pr create --title "PROJ-123: Short description" --body-file "$BODY_FILE"; then
  rm "$BODY_FILE"
else
  echo "PR creation failed. Body saved to: $BODY_FILE" >&2; false
fi
```

**View / list / comment**:
```bash
gh pr view 123
gh pr list
gh pr comment 123 --body "Comment text"
```

**CI**:
```bash
gh run list --branch feature/PROJ-123
gh run view 456 --log-failed
```

## GitLab (`glab`)

**Authenticate**: `glab auth login`

**Create MR** — `glab` has no `--description-file`; use `"$(cat file)"`:
```bash
BODY_FILE=$(mktemp /tmp/mr-body.XXXXXX)
cat > "$BODY_FILE" << 'EOF'
## Ticket
PROJ-123

## Summary
Brief description.
EOF

if glab mr create --title "PROJ-123: Short description" --description "$(cat "$BODY_FILE")"; then
  rm "$BODY_FILE"
else
  echo "MR creation failed. Body saved to: $BODY_FILE" >&2; false
fi
```

**View / list / comment**:
```bash
glab mr view 123
glab mr list
glab mr note 123 --message "Comment text"
```

**CI**:
```bash
glab ci list --branch feature/PROJ-123
glab ci trace 789
```

## Common git operations

**Branch** — see [references/branch-operations.md](references/branch-operations.md) for naming convention and rename procedure:
```bash
git checkout -b feature/PROJ-123-short-description
git push -u origin feature/PROJ-123-short-description
```

**Commit**:
```bash
git add path/to/file
git commit -m "PROJ-123: Short description"
git log --oneline -10
```

**Diff** — see [references/diff-practices.md](references/diff-practices.md) for full decision tree and pager guidance:
```bash
git --no-pager diff origin/main...HEAD
git --no-pager diff --name-only origin/main...HEAD
git --no-pager diff --stat origin/main...HEAD
git --no-pager diff origin/main...HEAD -- path/to/file.php
```

## TODO

- Project-specific branch protection rules may vary
- Custom CI/CD job names may vary
