# Enhancement Workflow

## Schema Changes (New Fields, Tables, Relationships)

### Process
1. **Branch in Dolt** — Create feature branch: `dolt checkout -b add-custom-field`
2. **Write migration** — SQL ALTER TABLE or CREATE TABLE in `migrations/YYYYMMDD_description.sql`
3. **Test on copy** — Fork Dolt branch, apply migration, verify existing data + queries still work
4. **Backfill if needed** — If adding required field to existing noun, write UPDATE script to populate from related data or defaults
5. **Merge to main** — `dolt commit` and merge when ready

### Existing Data Handling
- **Adding optional field** — No backfill needed, NULL default
- **Adding required field** — Must backfill; use UPDATE based on existing relationships
- **Changing field type** — Requires data migration (e.g., string → enum); test thoroughly
- **Adding relationship** — New FK column; backfill by linking to existing related records
- **Denormalization** — Add cached field (e.g., `Person.last_interaction_date`); write trigger or scheduled update script

## New CLI Verbs (New Functionality, Same Schema)

### Process
1. Define verb in CLI command registry
2. Write query/business logic (uses existing schema)
3. Test with `crm <verb> <args>`
4. No schema migration needed

### Examples
- `crm person activity Woody` — query Notes linked to Woody (existing tables, new query)
- `crm stale --days 90` — flag People with no recent Meeting/Task (new query, no schema change)
- `crm sync-calendar` — sync with Calendar.app (new logic, reads existing Meeting schema)

## New Claude Skills (Optional, Runtime Enhancements)

### Process
1. Define skill that reads/analyzes existing schema
2. Skills are **additive, not required** for core functionality
3. Work offline if skill unavailable

### Examples
- Skill: "Verify LinkedIn for meeting attendees" — reads Person.linkedin_url, queries LinkedIn, reports
- Skill: "Summarize person relationship" — reads Note/Task/Meeting linked to Person, synthesizes
- Skill: "Suggest next steps" — reads Opportunity stage + last interactions, recommends actions

## Priority Order for Enhancements

1. **Schema changes** (e.g., "add status to Organization") — affects data integrity, requires migration
2. **CLI verbs** (e.g., "crm task overdue") — uses existing schema, enables workflows
3. **Claude skills** (e.g., "summarize relationship") — nice-to-have, optional

## Rollback Strategy

- **Schema changes:** Dolt branch history; revert commit to undo
- **Data:** Full audit trail in Git; can see every change with `dolt log -p`
- **CLI verbs:** Just code changes; revert CLI file
- **Claude skills:** Disable skill without data impact

## Testing Workflow

```bash
# Create isolated branch
dolt checkout -b test/new-field
dolt sql < migrations/add_field.sql

# Test queries work
crm person list  # should still work
crm task --person X  # verify JOIN still works

# Verify backfill (if added required field)
dolt sql -q "SELECT COUNT(*) FROM task WHERE owner_id IS NULL"  # should be 0

# Merge when confident
dolt commit -am "Add new field"
dolt checkout main
dolt merge test/new-field
```
