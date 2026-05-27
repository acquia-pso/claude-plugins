---
name: acquia-cli
description: Acquia Cloud CLI (acli) for managing environments, databases, files, deployments, logs, env variables, cron and more.
---

# Acquia CLI Quick Reference

`$ACQUIA_APP_UUID` is set in `.claude/settings.json` — use it in all commands.

## Environment Identifier Pattern

All environment-scoped commands use `$ACQUIA_APP_UUID.<env>`:
- `$ACQUIA_APP_UUID.dev` — Development
- `$ACQUIA_APP_UUID.test` — Staging
- `$ACQUIA_APP_UUID.prod` — Production

## List Environments

```bash
acli api:environments:list --applicationUuid=$ACQUIA_APP_UUID
```

## View Environment Info

```bash
acli api:environments:info $ACQUIA_APP_UUID.dev
```

## Database Operations

**Pull database from dev to local**:
```bash
acli pull:db --no-interaction
```

**Options**:
- `--environment=dev` (default)
- `--database=default` (default)
- `--no-interaction` (skip prompts)

**Note**: This overwrites your local database. Always confirm before running.

## Files Operations

**Pull files from dev to local**:
```bash
acli pull:files --no-interaction
```

**Options**:
- `--environment=dev` (default)
- `--no-interaction` (skip prompts)

## Environment Variables

**List environment variables**:
```bash
acli api:environments:variable:list $ACQUIA_APP_UUID.dev
```

**Get specific variable**:
```bash
acli api:environments:variable:get $ACQUIA_APP_UUID.dev VARIABLE_NAME
```

**Set environment variable**:
```bash
acli api:environments:variable:create $ACQUIA_APP_UUID.dev VARIABLE_NAME "value"
```

## Code Deployment

**List code branches**:
```bash
acli api:applications:code:list $ACQUIA_APP_UUID
```

**Deploy branch to environment**:
```bash
acli api:environments:code-switch $ACQUIA_APP_UUID.dev --branch=feature-branch
```

## Domain Management

**List domains for environment**:
```bash
acli api:environments:domain:list $ACQUIA_APP_UUID.dev
```

## Logs

**View logs**:
```bash
acli api:environments:log:tail $ACQUIA_APP_UUID.dev apache-error
```

**Log types**:
- `apache-access`
- `apache-error`
- `drupal-request`
- `drupal-watchdog`

## Output Parsing

Most `acli` commands return JSON. Use `jq` to parse:

```bash
acli api:environments:list --applicationUuid=$ACQUIA_APP_UUID | jq '.[] | {name: .name, status: .status}'
```

## Safety Rules

- **Always confirm** before running write operations (`pull:db`, `pull:files`, `variable:create`)
- **Never interact with production** without explicit second confirmation
- **Read operations** (list, info, log) are safe and do not require confirmation

