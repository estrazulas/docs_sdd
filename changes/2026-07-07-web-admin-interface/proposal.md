## Why

HeadroomGate currently requires CLI commands (`headroom auth`, `headroom usage`) for all user management and usage monitoring operations. This creates friction for administrators and team leads who need to manage users, teams, and API keys quickly, and makes it difficult to visually explore usage patterns. A web-based admin interface removes this CLI dependency and provides an intuitive, role-aware management experience.

## What Changes

- New `/manage` route serving a multi-page admin web interface
- Session-based authentication via cookie httpOnly tokens (login with existing `hr_*` API keys)
- **User management**: create, list, and deactivate/reactivate users through a web UI
- **Team management**: create teams and assign users to teams
- **API key management**: create keys for developers, list active keys, revoke compromised keys — with copy-to-clipboard JSON output for distribution
- **Usage dashboard**: view per-user and per-team token consumption, drill into individual user sessions, search request history by term, and visualize top consumers via charts
- **RBAC enforcement**: team leads see only their team's data; admins have full access — the same permission model already implemented in `headroom.auth.store`
- Integration with existing Neo4j auth store and Qdrant usage store — no new data stores

## Capabilities

### New Capabilities

- `admin-auth-session`: Session-based authentication for the admin web interface using existing `hr_*` API keys, with httpOnly cookie tokens and server-side session management
- `user-management`: Create, list, activate, and deactivate users through the web UI, respecting the existing RBAC model (admin full access, team_lead team-scoped)
- `team-management`: Create teams and assign users to them via the web interface
- `api-key-management`: Generate, list, and revoke `hr_*` API keys with copy-to-clipboard distribution format
- `usage-monitoring`: Visual dashboard showing token consumption by user and team, session drill-down, full-text search across request history, and top-consumer charts

### Modified Capabilities

<!-- No existing specs to modify -->

## Impact

- **New files**: Admin FastAPI router (`headroom/admin/`), Jinja2 templates (`headroom/admin/templates/`), session management module
- **Existing code**: `headroom/auth/store.py` (Neo4jAuthStore) — reused as-is; `headroom/usage/store.py` (AuditStore) — reused as-is; `headroom/proxy/server.py` — light touch to register `/manage` routes and session middleware
- **Dependencies**: No new Python dependencies beyond what FastAPI and Jinja2 already provide in the project
- **Backward compatibility**: No **BREAKING** changes — the CLI and proxy API continue to work unchanged
- **Security**: httpOnly session cookies prevent XSS access; API keys never stored in browser localStorage; session TTL enforced server-side
