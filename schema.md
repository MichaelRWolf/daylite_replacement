# Daylite Replacement — Schema

**See [schema_diagram.md](schema_diagram.md) for visual ER diagram (Typora-compatible mermaid)**

## Core Nouns

### Person
- `id` (UUID)
- `name` (string)
- `email` (string)
- `phone` (string)
- `linkedin_url` (string)
- `organization_id` (FK → Organization)
- `notes` (JSON array of linked Note IDs)
- `created_at`, `updated_at` (timestamps)
- `status` (active/archived)

### Organization
- `id` (UUID)
- `name` (string)
- `website` (string)
- `industry` (string)
- `notes` (JSON array of linked Note IDs)
- `created_at`, `updated_at` (timestamps)
- `status` (active/archived)

### Note
- `id` (UUID)
- `content` (string, rich text)
- `linked_to` (JSON: {noun_type, noun_id} — can link to any noun)
- `created_at`, `updated_at` (timestamps)
- `tags` (string array)

### Meeting/Appointment
- `id` (UUID)
- `title` (string)
- `start_time` (ISO datetime)
- `end_time` (ISO datetime)
- `timezone` (string — person's TZ if 1:1)
- `attendees` (JSON array of Person IDs)
- `project_id` (FK → Project, optional)
- `opportunity_id` (FK → Opportunity, optional)
- `zoom_link` (string, optional)
- `notes` (JSON array of linked Note IDs)
- `created_at`, `updated_at` (timestamps)

### Task
- `id` (UUID)
- `title` (string)
- `description` (string)
- `owner_id` (FK → Person — who it's assigned to)
- `related_person_id` (FK → Person, optional — who it's about)
- `project_id` (FK → Project, optional)
- `opportunity_id` (FK → Opportunity, optional)
- `status` (pending/in-progress/done)
- `due_date` (date)
- `priority` (1-5)
- `created_at`, `updated_at` (timestamps)

### Opportunity
- `id` (UUID)
- `title` (string)
- `description` (string)
- `person_id` (FK → Person — primary contact)
- `organization_id` (FK → Organization)
- `stage` (enum: prospect/active/proposal/won/lost)
- `value` (decimal, optional)
- `expected_close_date` (date)
- `project_id` (FK → Project, optional)
- `notes` (JSON array of linked Note IDs)
- `created_at`, `updated_at` (timestamps)

### Project
- `id` (UUID)
- `name` (string)
- `description` (string)
- `status` (active/on-hold/completed/archived)
- `owner_id` (FK → Person)
- `team_ids` (JSON array of Person IDs)
- `related_opportunity_id` (FK → Opportunity, optional)
- `notes` (JSON array of linked Note IDs)
- `created_at`, `updated_at` (timestamps)

## Key Relationships

- Person ↔ Organization (many-to-one)
- Meeting ↔ Person (many-to-many via attendees)
- Task ↔ Person (assigned to/about)
- Task ↔ Project (belongs to)
- Task ↔ Opportunity (related to)
- Opportunity ↔ Person (primary contact)
- Opportunity ↔ Organization
- Opportunity ↔ Project
- Project ↔ Person (owner, team)
- Note ↔ Any noun (polymorphic link)

## Storage

**Backend:** Dolt (Git-based SQL) for version control and federation
**Schema:** SQL schema with JSON fields for flexibility
**Indexing:** Timestamps, status fields, foreign keys

## Queries (CLI layer)

- `crm person list` — all contacts
- `crm meeting today` — today's appointments
- `crm meeting 2:00 zoom` — get 2pm meeting zoom link
- `crm task overdue` — pending tasks past due
- `crm opportunity active` — active deals
- `crm note --person Woody` — all notes linked to Woody
- `crm task --project X --status pending` — project tasks

## Claude-Optional Queries

- Verify LinkedIn connections to everyone in today's meeting TODOs
- Summarize all notes from person X in last N days
- Flag stale relationships (no interaction in 90+ days)
- Suggest next steps based on opportunity stage
