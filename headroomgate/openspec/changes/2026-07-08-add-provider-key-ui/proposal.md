## Why

HeadroomGate requires CLI commands (`headroom auth set-provider-key`, `headroom auth list-provider-keys`) to manage provider API keys for each role. Administrators must leave the web interface to configure which provider keys a role can use. This creates friction when onboarding new teams or rotating keys, and it means the admin interface cannot complete the "create user → assign role → provide access" workflow end-to-end.

Adding a Roles page with per-role provider key management closes this gap, letting admins manage the full user lifecycle from within the web admin.

## What Changes

- New **Roles** page at `/manage/roles` — read-only list of all roles (name, description, provider keys)
- Per-role **provider key management** — set and remove API keys for Anthropic, OpenAI, Gemini directly from the UI
- Role sidebar navigation entry in the admin interface
- Backend API endpoints: `GET /manage/api/roles`, `GET /manage/api/roles/<name>/providers`, `POST /manage/api/roles/<name>/providers/<provider>`, `DELETE /manage/api/roles/<name>/providers/<provider>`
- New `remove_provider_key` method in `Neo4jAuthStore` (currently missing)
- RBAC enforcement: admin-only access for the Roles page (team leads manage users/keys within their team but do not control provider keys per role)
- Only the `admin` role is protected against provider key modifications via the web UI — if an admin needs to change their own role's keys, they use the CLI. All other roles (including base roles `developer`, `team_lead`, `viewer`) are fully manageable.

## Capabilities

### New Capabilities
- `role-provider-keys`: Web interface to view, set, and remove provider API keys (Anthropic, OpenAI, Gemini) per role, with encrypted storage and RBAC protection

### Modified Capabilities

(No existing spec requirement changes)

## Impact

- **New files**: `headroom/admin/templates/roles.html`
- **Modified files**:
  - `headroom/admin/templates/base.html` — add "Roles" sidebar link (visible to admin only)
  - `headroom/admin/router.py` — add role listing page route + provider key API endpoints
  - `headroom/auth/store.py` — add `remove_provider_key()` method
- **No CLI changes** — existing CLI commands continue working
- **No breaking changes** — the data format for `provider_keys` on the Role node stays unchanged (flat dict of provider → encrypted key, with simple removal)
- **E2E tests added** — `TestRolesE2E` class in `tests/test_admin/test_e2e.py` covers role list rendering, protected badge, Keys button visibility, modal interaction, set/remove provider key flow, and non-admin access denial (6 test cases)
