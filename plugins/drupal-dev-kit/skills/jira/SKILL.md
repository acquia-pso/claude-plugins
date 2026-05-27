---
name: jira
description: Jira CLI. View/create/update issues, sprints, backlog, transitions. Triggers on issue keys (ABC-123), tickets, jira keyword.
---

# Jira CLI Skill

Natural language interaction with Jira using the `jira` CLI.

## Installation

```bash
brew install ankitpokhrel/jira-cli/jira-cli
```

## Authentication

`jira init` sets up the server URL and project config. The API token is **not** set via `jira init` — it must be exported as an env var in your shell profile:

```bash
# In ~/.zshrc (or ~/.zprofile / ~/.bashrc)
export JIRA_API_TOKEN="your-atlassian-api-token"
export JIRA_AUTH_TYPE=basic
```

Generate a token at: https://id.atlassian.com/manage-profile/security/api-tokens

After adding/updating the token, reload your shell:
```bash
source ~/.zshrc
```

Then run `jira init` to configure the server URL and project:
```bash
jira init
```

---

## Quick Reference

| Intent | Command |
|--------|---------|
| View issue | `jira issue view ISSUE-KEY` |
| List my issues | `jira issue list -a $(jira me)` |
| My in-progress | `jira issue list -a $(jira me) -s "In Progress"` |
| Create issue | `jira issue create -t Type -s "Summary" --template /path/to/body.txt --no-input` |
| Create sub-task | `jira issue create -t Sub-task -s "Summary" --parent STORY-KEY --template /path --no-input` |
| Move/transition | `jira issue move ISSUE-KEY "State"` |
| Assign to me | `jira issue assign ISSUE-KEY $(jira me)` |
| Unassign | `jira issue assign ISSUE-KEY x` |
| Add comment | `jira issue comment add ISSUE-KEY --template /path/to/file --no-input` |
| Open in browser | `jira open ISSUE-KEY` |
| Current sprint | `jira sprint list --state active` |
| Who am I | `jira me` |

---

## Commands

### View a ticket

```bash
jira issue view PROJ-123
```

**Output**: Displays ticket details including title, description, status, assignee, labels, and linked issues.

### List tickets by project

```bash
jira issue list --project PROJ
```

**Options**:
- `--status "In Progress"` — filter by status
- `--assignee john.doe@example.com` — filter by assignee
- `--jql "sprint = 'Sprint 12'"` — custom JQL query

### List tickets in current sprint

```bash
jira sprint list --current --project PROJ
```

### Add a comment

**Always use `--template` + `--no-input`** — passing the comment body as an inline string causes two problems:
1. **argv limitations**: the full comment body is passed via command-line argument, which has length limits and can mangle whitespace/newlines if unquoted
2. **Interactive prompt**: without `--no-input`, the CLI shows a "What's next?" prompt that blocks non-interactive execution

```bash
# Safe pattern — write to temp file, use --template, suppress interactive prompt
COMMENT_FILE=$(mktemp /tmp/jira-comment.XXXXXX)
cat > "$COMMENT_FILE" << 'EOF'
Your comment text here.

Supports `backticks`, $variables, $(subshells), and "quotes" safely.
Multi-line content works too.
EOF
if jira issue comment add PROJ-123 --template "$COMMENT_FILE" --no-input; then
  rm "$COMMENT_FILE"
else
  echo "Failed to add comment. Temp file preserved at: $COMMENT_FILE" >&2
  false
fi
```

> **Why `--template`?** The file path is passed to the jira CLI directly — the shell never sees or expands the content. Combined with `--no-input`, this is fully non-interactive and safe for all content.

> **Why not `"$(cat file)"`?** Command substitution inserts the file's contents into the command line before `jira` runs. That does **not** cause the content itself to be re-evaluated for backticks, `$vars`, or `$(...)`, but it still means you're passing the full comment body via argv instead of as a file. `--template` is more reliable for multi-line or large content and avoids whitespace/newline and command-length pitfalls.

### Transition ticket status

```bash
jira issue move PROJ-123 "In Progress"
```

**Note**: Status names must match exactly as configured in Jira.

### Create a ticket

**Use `--template` for the body** — same escaping and interactivity issues apply to `--body "..."`:

```bash
# Safe pattern — write body to temp file, use --template
BODY_FILE=$(mktemp /tmp/jira-body.XXXXXX)
cat > "$BODY_FILE" << 'EOF'
Description here.

Supports `backticks`, $variables, and multi-line content safely.
EOF
if jira issue create --project PROJ --type Task --summary "Do the thing" --template "$BODY_FILE" --no-input; then
  rm "$BODY_FILE"
else
  echo "jira issue create failed; body template preserved at: $BODY_FILE" >&2
  false
fi
```

> **Note:** `--template` takes precedence over `--body` when both are provided. Use one or the other.

### Create a sub-task

Use `--type Sub-task` and `--parent STORY-KEY`. Using any other type (e.g. `Task`) with `--parent` fails unless the parent is an Epic.

```bash
BODY_FILE=$(mktemp /tmp/jira-body.XXXXXX)
cat > "$BODY_FILE" << 'EOF'
Sub-task description here.
EOF
if jira issue create --project PROJ --type Sub-task --summary "Do the thing" \
    --parent PROJ-123 --template "$BODY_FILE" --no-input; then
  rm "$BODY_FILE"
else
  echo "jira issue create failed; body template preserved at: $BODY_FILE" >&2
  false
fi
```

| Parent type | Correct child type |
|-------------|-------------------|
| Epic | Story, Task (via `--parent`) |
| Story | Sub-task (via `--parent`) |

### Assign ticket

```bash
# Assign to me
jira issue assign PROJ-123 $(jira me)

# Assign to specific user
jira issue assign PROJ-123 john.doe@example.com

# Unassign
jira issue assign PROJ-123 x
```

---

## Output Parsing

The `jira issue view` command returns structured output. Key fields:
- **Key**: Ticket ID (e.g., PROJ-123)
- **Summary**: Ticket title
- **Description**: Full description (may be long)
- **Status**: Current status (e.g., "In Progress", "Done")
- **Assignee**: Email of assigned user
- **Story Points**: If present (custom field)
- **Labels**: Comma-separated list
- **Linked Issues**: Related, blocks, blocked by

### Example Output

```
PROJ-123: Add user authentication

Type:        Task
Status:      In Progress
Assignee:    john.doe@example.com
Priority:    High
Labels:      security, authentication
Story Points: 5

Description:
Implement OAuth2 authentication flow for user login. Users should be able to log in with email/password, and tokens should be stored securely.

Linked Issues:
  Blocks: PROJ-124 (Profile page requires auth)
  Relates to: PROJ-100 (Security audit)
```

---

## Triggers

Use this skill when the user:
- Mentions "create a jira ticket"
- References an issue key (e.g., "show me PROJ-123")
- Asks to "list my tickets"
- Wants to "move ticket to done"
- Asks "what's in the current sprint"

---

## Issue Key Detection

Issue keys follow the pattern: `[A-Z]+-[0-9]+` (e.g., PROJ-123, ABC-1).
When a user mentions an issue key in conversation:
- Use `jira issue view KEY` or `jira open KEY`

---

## Workflow

### Creating tickets

1. Research context if user references code/tickets/PRs
2. Draft ticket content
3. Review with user
4. Create using appropriate backend

### Updating tickets

1. Fetch issue details first
2. Check status (careful with in-progress tickets)
3. Show current vs proposed changes
4. Get approval before updating
5. Add comment explaining changes

---

## Before Any Operation

Ask yourself:
1. **What's the current state?** — Always fetch the issue first. Don't assume status, assignee, or fields are what user thinks they are.
2. **Who else is affected?** — Check watchers, linked issues, parent epics. A "simple edit" might notify 10 people.
3. **Is this reversible?** — Transitions may have one-way gates. Some workflows require intermediate states. Description edits have no undo.
4. **Do I have the right identifiers?** — Issue keys, transition IDs, account IDs.

---

## NEVER

- **NEVER transition without fetching current status** — Workflows may require intermediate states. "To Do" → "Done" might fail silently if "In Progress" is required first.
- **NEVER edit description without showing original** — Jira has no undo. User must see what they're replacing.
- **NEVER use `--no-input` without all required fields** — Fails silently with cryptic errors. Check project's required fields first.
- **NEVER assume transition names are universal** — "Done", "Closed", "Complete" vary by project. Always get available transitions first.
- **NEVER bulk-modify without explicit approval** — Each ticket change notifies watchers. 10 edits = 10 notification storms.

---

## Authentication Troubleshooting

`jira me` succeeding but `jira issue view`, `jira issue list`, or `jira board list` returning 401/404 means the API token is stale or lacks the required scopes — even though identity resolution still works.

**Validate first — before attempting any issue operations:**

```bash
# Quick auth health check: identity + a real issue lookup
jira me && jira issue view PROJ-1 2>&1 | head -5
```

If `jira me` works but issue/board calls fail, the `JIRA_API_TOKEN` env var is stale. Guide the user:

```
Your JIRA_API_TOKEN has expired or lost board access.

To fix:
1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Create a new token
3. Find where JIRA_API_TOKEN is set in your shell:
     grep -n "JIRA_API_TOKEN" ~/.zshrc ~/.zprofile ~/.bashrc 2>/dev/null
4. Update the token value on that line, then reload:
     source ~/.zshrc   (or whichever file it's in)
5. Verify: jira issue view <ISSUE-KEY>
```

Note: `jira init` sets up the server/project config but does **not** prompt for the API token — the token must be set as the `JIRA_API_TOKEN` environment variable in your shell profile.

**Do not** attempt workarounds (REST API calls, keychain lookups, env var scraping) — surface the issue clearly and ask the user to update their token. Save any drafted content to a local file so no work is lost while the user fixes it.

---

## Safety

- Always show the command/tool call before running it
- Always get approval before modifying tickets
- Preserve original information when editing
- Verify updates after applying
- Always surface authentication issues clearly so the user can resolve them

---

## TODO

- Project-specific custom fields may need to be documented
- JQL query examples for project-specific workflows
