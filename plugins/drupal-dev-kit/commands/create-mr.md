---
name: create-mr
description: Push the current branch and create a PR/MR with a generated description
---

# Create MR Command

This command pushes the current branch to the remote repository and creates a PR/MR (Pull Request on GitHub, Merge Request on GitLab) with an auto-generated description.

## Execution Steps

1. **Check branch name**
   - Ensure current branch follows naming convention from `.claude/standards.md`
   - Expected format: `feature/PROJ-123-short-description` or `bugfix/PROJ-123-short-description`
   - Jira ticket ID required unless no ticket exists (internal/PoC work)
   - If branch name is invalid, warn and ask developer to rename first

2. **Push branch**
   ```bash
   git push -u origin <branch-name>
   ```

3. **Generate a concise PR/MR description**
   - Extract from the conversation: Jira ticket reference, summary of changes, testing steps
   - **Keep it short** — aim for ≤ 20 lines total; do not write exhaustive change logs
   - Use this template:
     ```markdown
     ## Ticket

     PROJ-123

     ## Summary

     One or two sentences describing what changed and why.

     ## Testing Steps

     1. Step one
     2. Step two
     3. Expected result
     ```

4. **Write the description to a temp file**

   > ⚠️ **Never pass the description inline** (`--body "..."` or `--body "$(cat file)"`).
   > Special characters (backticks, `$`, `!`, quotes, newlines) will be misinterpreted by the shell and cause the command to fail or produce a garbled description.
   > Always write to a file first and reference the file.

   ```bash
   BODY_FILE=$(mktemp /tmp/pr-body.XXXXXX)
   cat > "$BODY_FILE" << 'PREOF'
   <description content here — no variable expansion occurs inside a quoted heredoc>
   PREOF
   ```

5. **Create the PR/MR using the file**

   **GitHub**:
   ```bash
   if gh pr create --title "PROJ-123: Short description" --body-file "$BODY_FILE"; then
     rm "$BODY_FILE"
   else
     echo "PR creation failed. Body saved to: $BODY_FILE" >&2
     false
   fi
   ```

   **GitLab** (`glab` has no `--description-file`, so `$(cat)` is used — safe because the file was written via a quoted heredoc):
   ```bash
   if glab mr create --title "PROJ-123: Short description" --description "$(cat "$BODY_FILE")"; then
     rm "$BODY_FILE"
   else
     echo "MR creation failed. Body saved to: $BODY_FILE" >&2
     false
   fi
   ```

6. **Return URL and clean up**
   - Display the URL of the created PR/MR
   - Remind developer to review the description and edit if needed
   - Temp file is automatically cleaned up on success

## Title Format

Extract Jira ticket ID from branch name and use it in title:
- Branch: `feature/PROJ-123-add-auth`
- Title: `PROJ-123: Add user authentication`

## Description Length Guidelines

| Section       | Guideline                              |
|---------------|----------------------------------------|
| Ticket        | Ticket ID only (e.g. `PROJ-123`)       |
| Summary       | 1–3 sentences max                      |
| Testing Steps | 3–6 numbered steps                     |
| **Total**     | **≤ 20 lines**                         |

Do not include: full diffs, exhaustive file lists, implementation details, or anything that belongs in a commit message.

## Example Usage

User says: `/create-mr`

You respond:
1. Check branch name (e.g., `feature/PROJ-123-add-auth`)
2. Push branch: `git push -u origin feature/PROJ-123-add-auth`
3. Create temp file: `BODY_FILE=$(mktemp /tmp/pr-body.XXXXXX)`
4. Write concise description to `$BODY_FILE` using a quoted heredoc
5. Run: `gh pr create --title "PROJ-123: Add user authentication" --body-file "$BODY_FILE"`
6. Delete `$BODY_FILE` on success (or preserve on failure)
7. Display URL: `PR created: https://github.com/org/repo/pull/456`

## Fallback

If session context does not contain enough information for the description:
- Generate a minimal description with just the ticket reference and a placeholder summary
- Warn developer: "Please edit the PR/MR description manually to add summary and testing steps."

## TODO

- Project-specific PR/MR template locations may vary (`.github/pull_request_template.md`, `.gitlab/merge_request_templates/`)
- Custom description format may be required
