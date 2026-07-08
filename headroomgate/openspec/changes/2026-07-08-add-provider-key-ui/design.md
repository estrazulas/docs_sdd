## Context

The admin web interface at `/manage` uses FastAPI + Jinja2 + Alpine.js + Tailwind CSS (same stack as the existing dashboard). Each admin page follows a consistent pattern: a sidebar nav entry in `base.html`, a page route in `router.py`, API endpoints that delegate to `Neo4jAuthStore`, and a Jinja2 template with Alpine.js for interactivity.

Provider API keys are stored encrypted (Fernet) as a JSON dict on each `:Role` Neo4j node in `r.provider_keys`. The auth middleware (`provider_injector.py`) decrypts the relevant key at runtime and injects it into the upstream request.

Currently there is no web UI for role or provider key management — it's CLI-only (`headroom auth set-provider-key`, `headroom auth list-provider-keys`). The store has `set_provider_key`, `list_provider_keys`, `get_provider_key`, `list_roles`, and `create_role` but no `remove_provider_key` method.

## Goals / Non-Goals

**Goals:**
- Admin-only "Roles" page listing all roles with name and description
- Per-role provider key viewer (shows which providers are configured)
- Set a provider key (Anthropic, OpenAI, Gemini) for a custom role via modal form
- Remove a provider key from a custom role with confirmation
- Base roles (admin, developer, team_lead, viewer) are protected — no provider key modification allowed
- Consistent look & feel with existing admin pages (Alpine.js modals, Tailwind tables)

**Non-Goals:**
- Deleting or creating roles from the web UI (CLI-only for now)
- Editing role descriptions or names from the web UI
- Provider key validation on the web UI (CLI validates with a test request; web UI stores as-is)
- Team lead access to roles page (admin-only — team leads manage users/keys within their scope)
- Bulk operations on provider keys

## Decisions

### 1. UI Pattern: Role list table + provider key drawer

**Choice**: A two-column card list showing all roles, with an eye icon on each row that opens a provider key management modal (list → set → remove).

**Rationale**: Roles are few (typically 4-10), so a card grid similar to the Teams page works well. The provider key management is per-role, so a modal avoids navigation complexity. Matches the existing modal pattern (users page has a create-user modal, teams page has add-user modal).

**Flow**:
1. Role list page loads → `GET /manage/api/roles` → renders card grid
2. Admin clicks gear icon on a role → modal opens → `GET /manage/api/roles/<name>/providers` → shows current providers
3. Admin clicks "Add Key" → inline form within modal → selects provider dropdown (anthropic, openai, gemini) + password input → `POST /manage/api/roles/<name>/providers/<provider>` with encrypted key
4. Admin clicks remove icon (trash) on a configured provider → confirmation → `DELETE /manage/api/roles/<name>/providers/<provider>` → key removed from JSON dict

### 2. API Endpoint Design

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/manage/api/roles` | List all roles | admin only |
| GET | `/manage/api/roles/{name}/providers` | List configured providers for role (without key values) | admin only |
| POST | `/manage/api/roles/{name}/providers/{provider}` | Set provider key (Fernet-encrypted) | admin only |
| DELETE | `/manage/api/roles/{name}/providers/{provider}` | Remove provider key from role | admin only |

Following the same pattern as existing `/manage/api/teams`, `/manage/api/keys`, etc.

### 3. Admin role protection (client + server)

**Choice**: Server-side enforcement + client-side UI disabling.

**Rationale**: Only the `admin` role is protected from modifications via the web UI — if an admin needs to change their own role's provider keys, they use the CLI (`headroom auth set-provider-key`). All other roles (`developer`, `team_lead`, `viewer`, and custom roles) are fully editable via the web UI. Server-side check returns 403 for set/delete operations on the `admin` role. Client-side, the gear icon is hidden and a "Protected" badge is shown for the admin role only.

### 4. No provider key validation on web UI

**Choice**: The web UI does NOT validate the provider key with a test request.

**Rationale**: The CLI validates via `_validate_provider_key()` which makes a live HTTP request to the provider. Doing this from the web UI would add latency and expose provider-specific error details to the browser. The key is stored encrypted regardless — validation is a nice-to-have, not a security requirement. The admin can use the CLI to validate if unsure.

### 5. `remove_provider_key` store method

**Choice**: New method in `Neo4jAuthStore` that removes a provider entry from the `provider_keys` JSON dict.

**Rationale**: Currently `set_provider_key` encrypts and upserts. There is no way to remove a key without overwriting the entire dict. The new method will:
1. Fetch the existing `provider_keys` JSON from the Role node
2. Remove the specified provider entry
3. Write back the updated JSON
4. Return the updated list

### 6. Fernet encryption key availability

**Choice**: The web UI endpoint for `POST` requires `HEADROOM_ENCRYPTION_KEY` to be configured (same as CLI `set-provider-key`).

**Rationale**: The key must be encrypted before storage. If the env var is missing, return a 400 error telling the admin to configure it via `headroom auth generate-key` and set `HEADROOM_ENCRYPTION_KEY`.

## Risks / Trade-offs

- **Fernet key missing at runtime** → Mitigation: the POST endpoint checks `HEADROOM_ENCRYPTION_KEY` availability and returns a clear error message guiding the admin to run `headroom auth generate-key`
- **No key validation on web UI** → Admin may type a wrong/expired key that only fails at runtime. Acceptable: the provider_injector already returns a 502 with a clear error message when the upstream rejects the key. The admin can always test via CLI.
- **Removing a key is irreversible** → Mitigation: confirmation dialog before DELETE. The encrypted value is lost once removed — there's no "undo" without re-entering.
- **Provider name mismatch** → Mitigation: the UI presents a fixed dropdown (Anthropic, OpenAI, Gemini) rather than a free-text field. The dropdown values match the provider names expected by `provider_injector.py::PROVIDER_PATH_MAP`.
