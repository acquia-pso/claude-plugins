# Git Branch Operations

## Branch Naming Convention

Format: `<type>/PROJ-123-short-description`

| Prefix | Use for |
|--------|---------|
| `feature/` | New functionality |
| `bugfix/` | Bug fixes |
| `hotfix/` | Urgent production fixes |
| `chore/` | Maintenance, tooling, config |

```bash
git checkout -b feature/PROJ-123-short-description
git push -u origin feature/PROJ-123-short-description
```

## Renaming a Branch (Local + Remote)

> ⚠️ **The workflow below renames the branch by pushing `new-name` and deleting the old remote ref (`git push origin :old-name new-name`). Deleting the old head branch on GitHub/GitLab can automatically close an open PR that uses `old-name` as its head — even if you immediately push the new branch name. Always verify PR state after this workflow and re-open or recreate the PR if needed. If you need to preserve an existing PR/review history, prefer the hosting platform's server-side branch rename UI when available.**

> ⚠️ **After this delete-and-push workflow, `gh pr edit` can succeed silently on a closed PR, giving false confidence that the PR is still active. Always check PR state with `gh pr list` — not by attempting an edit.**

```bash
# 1. Rename locally
git branch -m old-name new-name

# 2. Push new name and delete old name in one command
git push origin :old-name new-name

# 3. Update local tracking
git branch --set-upstream-to=origin/new-name new-name

# 4. ALWAYS verify PR state after rename
gh pr list --state all | grep old-name   # confirm old PR is now closed
gh pr list --state open                  # confirm no open PR exists for this work

# 5. If the PR was closed, open a new one
BODY_FILE=$(mktemp /tmp/pr-body.XXXXXX)
cat > "$BODY_FILE" << 'EOF'
Re-opening PR after renaming branch from old-name to new-name.
EOF
gh pr create --title "PROJ-123: short description" --body-file "$BODY_FILE" --base main --head new-name \
  && rm "$BODY_FILE" \
  || echo "PR creation failed. Body saved to: $BODY_FILE" >&2
```

## Common Branch Commands

**List all branches**:
```bash
git branch -a
```

**Delete local branch**:
```bash
git branch -d feature/PROJ-123-short-description
```

**Delete remote branch**:
```bash
git push origin --delete feature/PROJ-123-short-description
```

**Check out a remote branch**:
```bash
git checkout -b feature/PROJ-123 origin/feature/PROJ-123
```
