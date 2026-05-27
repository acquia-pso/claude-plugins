---
name: sync-env
description: Sync local environment with Acquia dev (pull DB, import config, run updates)
---

# Sync Environment Command

This command syncs the local DDEV environment with the Acquia dev environment by pulling the database and running the standard deployment workflow.

## Execution Steps

1. **Check DDEV status**
   - Run `ddev status` to check if DDEV is running
   - If not running, start it: `ddev start`

2. **Invoke acquia-env-manager to pull DB**
   - Pull database from Acquia dev environment: `acli pull:db --no-interaction`
   - **Important**: This will overwrite the local database
   - Confirm with developer before proceeding: "This will replace your local database with dev. Proceed?"

3. **Run deployment workflow**
   - `ddev drush deploy` (runs config import, updates, cache rebuild)
   - This command executes:
     - `drush updatedb -y` (run database updates)
     - `drush config:import -y` (import config)
     - `drush cache:rebuild` (clear cache)

4. **Report success or errors**
   - Display summary of what was synced
   - Report any errors encountered
   - Suggest next steps (e.g., manual testing, clearing cache)

## Output Format

```
## Environment Sync Complete

✓ DDEV started
✓ Database pulled from dev environment
✓ Deployment workflow executed:
  - Database updates: 3 applied
  - Config imported: 12 changes
  - Cache cleared

Next steps:
1. Test your local site: ddev launch
2. Verify changes in your browser
```

## Error Handling

### DDEV Not Running
- Start DDEV automatically: `ddev start`
- Wait for it to finish starting before proceeding

### Database Pull Fails
- Check Acquia credentials: `acli auth:login`
- Verify `ACQUIA_APP_UUID` is correct
- Report error to developer with suggested fix

### Deployment Fails
- Report which step failed (updatedb, config import, cache rebuild)
- Display error message
- Suggest manual fix or rollback

## Safety

- Always confirm before overwriting local database
- Do not pull from production (use dev environment only)
- If developer declines confirmation, abort the sync

## Example Usage

User says: "/sync-env"

You respond:
1. Check DDEV status
2. Ask: "This will replace your local database with dev. Proceed? [y/N]"
3. If yes:
   - Invoke acquia-env-manager to pull DB
   - Run `ddev drush deploy`
   - Report results

## TODO

- Project-specific deployment commands may vary
- Custom sync scripts may be needed (e.g., pulling files, syncing media)
